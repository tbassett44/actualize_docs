---
title: /api/class/[schema].php
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
Below is an example [schema].php (for the deal object, so deal.php).

DEAL::handleRequest() is called automatically when the endpoint called is: <https://api.actualize.earth/core/module/deal/[load|feed|promotioncost]>

ALL FEEDs of data should be handled in the [schema].php field.  These requests should be guarded to ensure nobody gets to load feeds of data without some type of query selecting based on user scope/permissions/filtering

DEAL::checkPermissions($r,$schema,$last,$proposed,$type) is the permission checker.  This passes the $last data object as well as the $proposed data object.  This is useful to ensure changes are valid (eg not changing the author of a post).  It is also used to ensure an author can edit a post.  
$r is the request object. This contains most all relevant information you will need from the request, notably, $r['auth]['uid'] is set when the request has been authenticated based on token/app_id in the parameters. 

```php
 <?php
	class DEAL{
		public static $id='';
		public static function checkPermissions($r,$schema,$last,$proposed,$type){
			if($proposed['page']['id']==$r['auth']['uid']) return true;
			$page=db2::findOne(DB,'page',['id'=>$proposed['page']['id']]);
			if($page['admins']&&in_array($r['auth']['uid'],$page['admins'])) return true;
			return false;
		}
		public static function handleRequest($r){
			if(isset($r['path'][5])) self::$id=$r['path'][5];
			switch ($r['path'][4]) {
				case 'load':
					$out=self::load($r);
				break;
				case 'feed':
					$out=self::feed($r);
				break;
				case 'promotioncost':
					$out=self::promotionCost($r);
				break;
			}
			if(!isset($out)) $out['error']='no_data';
			return $out;
		}
		public static function feed($r){
			include_once(ROOT.'/api/class/formbuilder.php');
			$qs=formbuilder::getBasicRequest($r,'deal','feed');
			if(!self::$id) return ['error'=>'no_route'];
			if(self::$id=='general'){//placeholder for user_loc
				if(isset($r['qs']['geoloc'])) $qs['geoloc']=$r['qs']['geoloc'];
			}else if(self::$id=='mine'){
				$qs['query']['user']=$r['auth']['uid'];
			}else{
				$qs['query']['page.id']=self::$id;
			}
			//if(isset($r['qs']['query'])){//guard query!
			//	$allowed=['tag'];
			//	foreach($allowed as $k=>$v){
			//		if(isset($r['qs']['query'][$v])) $qs['query'][$v]=$r['qs']['query'][$v];
			//	}
			//}
			return formbuilder::feed(array(
				'auth'=>$r['auth'],
				'qs'=>$qs
			));
		}
		public static function currentPromotionRate(){
			return 100;
		}
		public static function promotionCost($r){
			$d=phi::ensure($r,['current']);
			if(!isset($d['current']['start'])){
				return ['error'=>'Must have a start Set'];
			}
			if(!isset($d['current']['end'])){
				return ['error'=>'Must have a end Set'];
			}
			if(!isset($d['current']['location'])){
				return ['error'=>'Must have a location set'];
			}
			$time_in_seconds=$d['current']['end']-$d['current']['start'];
			$rate=self::currentPromotionRate();
			$persecond=$rate/(60*60*24);
			$total=$cost=floor($time_in_seconds*$persecond);
			//$d['current']['radius']=(int) $d['current']['radius'];
			//get current point balance if trying to use points
			$points=db2::findOne(DB,'points',array('id'=>$r['auth']['uid']));
			$points=phi::keepFields($points,['balance']);
			if(!isset($points['balance'])) $points['balance']=0;			if(isset($r['qs']['current']['points'])&&(int)$r['qs']['current']['points']){
				$usePoints=(int) $r['qs']['current']['points'];
				if($usePoints>$points['balance']) return ['error'=>'You do not have enough points to use '.$usePoints.'. You only have '.$points['balance'].' point(s)'];
				$points['discount']=($usePoints*10);
				$total=$total-$points['discount'];//convert to cents
				$points['used']=$usePoints;
				if($total<0) $total=0;//because of rounding / point usage
			}
			return [
				'success'=>true,
				'data'=>[
					'cost'=>$cost,
					'total'=>$total,
					'rate'=>$rate,
					'points'=>$points,
					'time_in_days'=>$time_in_seconds/(60*60*24),
					'quote'=>'$1/day'
				]
			];
		}
		public static function preProcessPromotion($r,$d,$key,$opts){

			if(isset($d['current']['location']['data']['point'])){
				$d['current']['point']=$d['current']['location']['data']['point'];
			}
			$d['current']['payment']['description']='Payment for deal promotion for '.$d['current']['page']['data']['name'];
			$time_in_seconds=$d['current']['end']-$d['current']['start'];
			$d['current']['price']=$rate=self::currentPromotionRate();
			$persecond=$rate/(60*60*24);
			$cost=$time_in_seconds*$persecond;
			$d['current']['total']=floor($cost);
			//apply point discount
			if(isset($d['current']['points'])){
				$points=db2::findOne(DB,'points',array('id'=>$r['auth']['uid']));
				$points=phi::keepFields($points,['balance']);
				if(!isset($points['balance'])) $points['balance']=0;			
				if($d['current']['points']>$points['balance']) API::toHeaders(['error'=>'You do not have enough points to use '.$d['current']['points'].'. You only have '.$points['balance'].' point(s)']);
				$discount=($d['current']['points']*10);
				$d['current']['total']=$d['current']['total']-$discount;//convert to cents
				//$points['used']=$usePoints;
				if($d['current']['total']<0) $d['current']['total']=0;//because of rounding / point usage
				//do the actual point transfer and ensure it works!
				formbuilder::$auth=$r['auth'];//maybe this should happen globally
				$res=formbuilder::update('exchange',[
					'to'=>[
						'type'=>'page',
						'id'=>ACTUALIZE_UID
					],
					'from'=>[
						'type'=>'user',
						'id'=>$r['auth']['uid']
					],
					'amount'=>$d['current']['points'],
					'message'=>$d['current']['payment']['description'],
					'seed'=>1
				]);
				if(isset($res['error'])) API::toHeaders($res);
				//otherwise the transaction went through, add the reference to the payment object
				$d['current']['payment']['exchange_id']=$res['data']['id'];
				//die(json_encode($res));
				// die(json_encode($d['current']));
			}
			//if(!phi::$conf['prod']) $d['current']['total']=100;
			return $d;
		}
		public static function getFilter($r,$q){
			if($q&&sizeof($q)){
				$tq=[
					'$and'=>[
						['$or'=>[
							['expires'=>['$exists'=>false]],
							['expires'=>['$gte'=>time()]]
						]]
					]
				];
				//die(json_encode($q));
				foreach($q as $k=>$v){
					$tq['$and'][]=[$k=>$v];
				}
			}else{
				$tq=['$or'=>[
					['expires'=>['$exists'=>false]],
					['expires'=>['$gte'=>time()]]
				]];
			}
			return $tq;
		}
		public static function load($r){
			$c=db2::findOne(phi::$conf['dbname'],'deal',array('id'=>self::$id));
			$schema=core::getSchema('deal');
			$c=db2::graphOne(phi::$conf['dbname'],$c,$schema['graph']);
			//add in other data, like comments
			//check permissions, assume public for now
			return array('success'=>true,'data'=>$c);
		}
	}
?>
```
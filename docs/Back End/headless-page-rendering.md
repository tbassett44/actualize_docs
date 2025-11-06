---
title: Headless Page Rendering
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
Useful for generating tickets as PDFs or Flyers as images, puppeteer is used to spin up a headless browser that can load an internal webpage to be rendered as an image or pdf.

[Read Puppeteer documentation here](https://pptr.dev/) 

### Example as a PNG (for QR code generation, 300x300 image)

```javascript
// const phantom = require('phantom');
var tools=require('./tools.js'); //tools are common node 
tools.init(); //initialize tools
var fs=require('fs');
var opts=tools.getBase64(process.argv[2]);  //get the URL passed to the function
const puppeteer = require('puppeteer');
var path='/var/www/.cache/puppeteer/chrome/linux-121.0.6167.85/chrome-linux64/chrome';
if(tools.settings.isdev){
  path='/var/www/.cache/puppeteer/chrome/linux-119.0.6045.105/chrome-linux64/chrome';
}
(async () => {

  // Create a browser instance
  const browser = await puppeteer.launch({
    executablePath: path,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });

  // Create a new page
  const page = await browser.newPage();

  // Website URL to export as pdf
  const website_url = opts.url;

  // Open URL in current page
  await page.goto(website_url, { waitUntil: 'networkidle0' }); 

  //To reflect CSS used for screens instead of print
  await page.emulateMediaType('screen');

// Downlaod the PDF
  const pdf = await page.screenshot({
    path: opts.out,
    type:'png',
    clip:{
      x:0,
      y:0,
      width:300,
      height:300
    }
  });
  //console.log('wrote to: '+opts.out);
  // Close the browser instance
  await browser.close();
  process.exit();
})();
```

### Example PDF (for tickets)

```javascript
// const phantom = require('phantom');
var tools=require('./tools.js');
tools.init();
var fs=require('fs');
var opts=tools.getBase64(process.argv[2]);
const puppeteer = require('puppeteer');
var path='/var/www/.cache/puppeteer/chrome/linux-121.0.6167.85/chrome-linux64/chrome';
if(tools.settings.isdev){
  path='/var/www/.cache/puppeteer/chrome/linux-119.0.6045.105/chrome-linux64/chrome';
}
(async () => {

  // Create a browser instance
  const browser = await puppeteer.launch({
    executablePath: path,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });

  // Create a new page
  const page = await browser.newPage();

  // Website URL to export as pdf
  const website_url = opts.url;

  // Open URL in current page
  await page.goto(website_url, { waitUntil: 'networkidle0' }); 

  //To reflect CSS used for screens instead of print
  await page.emulateMediaType('screen');

// Downlaod the PDF
  const pdf = await page.pdf({
    path: opts.out,
    margin: { top: '100px', right: '50px', bottom: '100px', left: '50px' },
    printBackground: true,
    format: 'A4',
  });
  //console.log('wrote to: '+opts.out);
  // Close the browser instance
  await browser.close();
})();
```

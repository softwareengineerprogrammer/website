---
title: Enrichment SWU calculator
description: > 
  Interactive nuclear enrichment and SWU calculator where
  you can enter a given feed or product mass and a product
  enrichment and it will tell you how much feed and SWU you
  need. Also plots results interactively as you go.
author: nick
layout: default
date: 2022-10-27
image: /img/enrichment-ss.png
---
<div class="row">
<div class="col-md-12" markdown="1">

Use this to calculate how much natural uranium and separative work you need to get
a given amount of enriched uranium at a certain enrichment, and so on. Learn more about [isotopes here]({% link isotopes.md %}).

<div class="row">
<div class="col-md-2 text-nowrap" markdown="1">
Tails assay (wt %)
</div>
<div class="col-md-1 text-nowrap" markdown="1">
<input type="text" value="0.25" placeholder="Tails" class="form-control" id="tails-assay">
</div>
<div class="col-md-2">Feed assay (wt %)</div>
<div class="col-md-1"> 
<input type="text" value="0.711" placeholder="Feed assay" class="form-control" id="feed-assay">
</div>
</div>

<div class="row">
<label for="enrich" class="form-label">Product Enrichment (wt %)</label>
<div class="col-md-4" markdown="1">
<input type="range" value="4.0" step="0.1" class="form-range" id="enrich">
</div>
<div class="col-md-2" markdown="1">
<input type="text" value="4.0" class="form-control" id="enrich-text">
</div>
<div class="col-md-4" markdown="1">
<select class="form-select" aria-label="Presets" id="enrich-select">
  <option value="" selected>Choose Enrichment Preset</option>
  <option value="0.711">Natural Uranium (0.711%)</option>
  <option value="4.0">LEU (4%)</option>
  <option value="20.0">HALEU (20%)</option>
  <option value="90.0">Weapons-grade (90%)</option>
</select>
</div>
<div class="col-md-2" markdown="1" id="enrich-label">
LEU
</div>
</div>
<hr/>
<div class="form-check">
  <input class="form-check-input" type="radio" name="mode" value="0" id="productConstant">
  <label class="form-check-label" for="productConstant">
    Compute product mass from given feed mass
  </label>
</div>
<div class="form-check">
  <input class="form-check-input" type="radio" name="mode" value="1" id="inputConstant" checked>
  <label class="form-check-label" for="inputConstant">
    Compute feed mass from given product mass
  </label>
</div>

<div class="row">
<div class="col-md-3">
<label for="feed-mass" class="form-label">Feed mass (kg or tonne)</label>
<input type="text" value="" placeholder="Feed mass" class="form-control" id="feed-mass">
</div>
<div class="col-md-3">
<label for="product-mass" class="form-label">Product mass (kg or tonne)</label>
<input type="text" value="1" placeholder="Product mass" class="form-control" id="product-mass">
</div>
<div class="col-md-3">
<label for="tails-mass" class="form-label">Tails mass (kg or tonne)</label>
<input type="text" value="" placeholder="Tails mass" class="form-control" id="tails-mass" readonly>
</div>
<div class="col-md-3">
<label for="swu-mass" class="form-label">SWU (kg-SWU or tonne-SWU)</label>
<input type="text" value="" placeholder="SWUs" class="form-control" id="swu-mass" readonly>
</div>
</div>

<div class="row">
<div class="col-12" id='plot'> </div>
</div>

<div class="row">
<div class="col-12">
<p class="text-center">
<button id="save" class="btn btn-primary">
<i class="fa fa-clipboard fa-lg"></i> Copy
current settings to clipboard</button></p>
 </div>
</div>



<script src='https://cdn.plot.ly/plotly-2.16.1.min.js'></script>

<script>


let range=document.getElementById("enrich");
let number=document.getElementById('enrich-text')
let select=document.getElementById('enrich-select')
let tails_assay=document.getElementById('tails-assay')
let feed_assay=document.getElementById('feed-assay')
let feed_mass=document.getElementById('feed-mass')
let product_mass=document.getElementById('product-mass')
let tails_mass=document.getElementById('tails-mass')
let swu=document.getElementById('swu-mass')
let mode=document.getElementById('mode')
let prod_const=document.getElementById('productConstant')
let save=document.getElementById('save')

tails_assay.addEventListener("input",(e)=>{
  computeFeed();
})
feed_assay.addEventListener("input",(e)=>{
  computeFeed();
})
product_mass.addEventListener("input",(e)=>{
  computeFeed();
})
feed_mass.addEventListener("input",(e)=>{
  computeFeed();
})
range.addEventListener("input",(e)=>{
    let val = parseFloat(e.target.value);
    if (val<parseFloat(feed_assay.value)) {
        return
    }
  number.value=val;
  select.value=""
  setRange(val);
  computeFeed();
})
number.addEventListener("input",(e)=>{
  range.value=e.target.value;
  select.value=""
  setRange(e.target.value);
  computeFeed();
})
select.addEventListener("change",(e)=>{
    if (!e.target.value) {
        return;
    }
    number.value=e.target.value;
    range.value=e.target.value;
    setRange(e.target.value);
    computeFeed();
})

save.addEventListener("click", (e) => {
  copySettingsToClipboard();
})

function setRange(val) {
    let label=document.getElementById("enrich-label");
    if (val>=0.0 && val<0.7) {
        label.innerHTML="Depleted";
    }
    else if (val==0.7) {
        label.innerHTML="Natural";
    }
    else if (val>0.7 && val<=5) {
        label.innerHTML="LEU";
    }
    else if (val>5 && val<=20.0) {
        label.innerHTML="HALEU";
    }
    else if (val>20.0 && val<=99.0) {
        label.innerHTML="HEU";
    }
    else if (val>99.0 && val<=100.0) {
        label.innerHTML="😲";
    }
    else {
        label.innerHTML="Impossible";
    }
}


function computeFeed() {
    // Compute feed and SWU given desired product and enrichment
    // compute MF/MP = feed factor
    let mode = document.querySelector('input[name="mode"]:checked').value
    let feed_factor = (number.value - tails_assay.value)/(feed_assay.value-tails_assay.value)
    feed_factor= feed_factor;
    if (mode=="1") {
        feed_mass.value = (feed_factor * product_mass.value).toFixed(3);
    }
    else {
        product_mass.value = (feed_mass.value/feed_factor).toFixed(3);
    }
    // swufactor= SWU/MP
    let vxt = vx(tails_assay.value)
    let swu_factor=(vx(number.value) - vxt) -  feed_factor*(vx(feed_assay.value)-vxt)
    swu.value = (swu_factor * product_mass.value).toFixed(3);
    tails_mass.value = (feed_mass.value - product_mass.value).toFixed(3);

    // update plot data
    let u238 = {
        x: ['Feed', 'Product', 'Tails'],
        y: [feed_mass.value*(1-feed_assay.value/100.0), 
            product_mass.value*(1-number.value/100.0), 
            tails_mass.value*(1-tails_assay.value/100.0)
        ],
        name: 'U238',
        type: 'bar'
    };
    let u235 = {
        x: ['Feed', 'Product', 'Tails'],
        y: [feed_mass.value*(feed_assay.value/100.0), 
            product_mass.value*(number.value/100.0), 
            tails_mass.value*(tails_assay.value/100.0)
        ],
        name: 'U235',
        type: 'bar'
    };

    var data=[u238, u235];
    Plotly.react('plot', data, layout);
    

}

function vx(x) {
    // value function: gotta convert percentages to fraction
    let xn = Number(x/100.0);
    return (1.0-2*xn) * Math.log((1.0-xn)/xn);

}

function setInputVals() {
  // These can all be passed in as query params
  const input = new URLSearchParams(window.location.search);
  tails_assay.value = input.get("tails-assay") || 0.25;
  feed_assay.value = input.get("feed-assay") || 0.711;
  number.value = input.get("enrich") || 4.0;
  range.value=number.value;
  feed_mass.value = input.get("feed-mass") || 0;
  product_mass.value = input.get("product-mass") || 1;
  feed_mass.value = input.get("feed-mass") || 1;
  if (input.get("mode") == "0") {
      // check a checkbox
      prod_const.checked=true
  };
}

function copySettingsToClipboard() {
  let params = new URLSearchParams([
    ["tails-assay", tails_assay.value], 
    ["feed-assay", feed_assay.value],
    ["enrich", number.value],
    ["feed-mass", feed_mass.value],
    ["product-mass", product_mass.value],
    ["mode", prod_const.checked ? "0" : "1"],
  ]);

  let text = new URL(`${location.protocol + '//' + location.host + location.pathname}?${params}`);
  navigator.clipboard.writeText(text);
}

var trace1 = {
  x: ['Feed', 'Product', 'Tails'],
  y: [20, 14, 23],
  name: 'U238',
  type: 'bar'
};

var trace2 = {
  x: ['Feed', 'Product', 'Tails'],
  y: [12, 18, 29],
  name: 'U235',
  type: 'bar'
};

var data = [trace1, trace2];

var layout = {barmode: 'stack', yaxis: {
    type: 'log', autorange: true, 
}, 
autosize: true,
  margin: {
      t: 0
    },
};

Plotly.newPlot('plot', data, layout);

window.onresize = function() {
    Plotly.relayout('plot', {})
}

// compute initial values
setInputVals();
computeFeed();


</script>

</div>
</div>



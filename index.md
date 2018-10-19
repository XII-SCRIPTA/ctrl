---
layout: default
title: Interpretting Non-Literal Language Use in the Built Environment
custom_js:
- assets/js/index.js
custom_css:
- assets/css/index.css
---

<div id="header">
  <h1 id='title'>Interpretting Non-Literal Language Use in the Built Environment</h1>
  <hr class='edition' />
  <span class="authors">XII-SCRIPTORUM</span>
</div>

<br />
Voice and text based interaction have become the de facto standard for controlling internet connected appliances and devices in domestic settings Despite their popularity, these systems are subject to a number of limitations, most notably, their inability to interpret the subtleties of meaning in everyday language use. This thesis aims to advance the current understanding of how utterances are interpreted in the context of built environmets, especially as they relate to vague and ambiguous commands. In particular, we argue that in order to make progress in this domain, it is necessary to develop a common framework for knitting together two core components of pragmatic language understanding: <b>contextual information</b>, and <b>intutitive models</b> of how agents plan and carry out goals in environments. We demonstrate how these components can be used to audit the behavior of the system, enabling it to give reasoned explanations for why it behaved the way it did.

<img src="../assets/img/ToM.png" alt="Fig. 1: Graphical representation of the Bayesian RSA model." style="width: 400px;"/>
<center>Fig. 1: Change to wiring diagram.</center>


<div id='left'>

<h3>Accompanying Thesis</h3>
XII-SCRIPTORUM (2018). <i>A Bit cold, Isn't it? Interpretting Non-Literal Language Use in the Built Environment With Bayesian Theory of Mind</i>. xxxx, xxxx <a id="toggle-bibtex" href="#">[bibtex]</a>

<pre id="bibtex">
@mastersthesis{XII-SCRIPTORUM:18,
abstract = {},
author = {XII-SCRIPTORUM},
month = {october},
school = {xxxx},
title = {% raw %}{{A Bit cold, Isn't it? Interpretting Non-Literal Language Use in the Built Environment With Bayesian Theory of Mind.}}{% endraw %},
year = {2018}
}
</pre>

<h3>Local Installation Instructions</h3>

<p> All instructions are for Mac OS version <code>10.13.6</code> onwards. WebPPL is hosted locally using the <a href="https://nodejs.org/en/"> node.js</a>) runtime environment. The <b>node package manager</b> (npm) repository is used to download and install WebPPL. Running <code>npm install -g webppl</code> in a terminal installs the latest version of WebPPL from npm. Once installed, the webppl program <code>foo</code> can be run globally in the terminal using the <code>webppl</code> prefix, i.e. <code>webppl foo.webppl</code> </p>

<p>A number of WebPPL specific packages and external javascript libraries are required for running the code locally. In the root or home directory (<code>cd ~</code>), create a new folder (<code>mkdir .webppl</code>). From here, install the <b>webppl-viz</b> library using the following command: <code>npm install probmods/webppl-viz</code>. </p>




</div>

{% assign sorted_pages = site.pages | sort:"name" %}

<div id="right">

<h3>Chapters</h3>
 
<ol>
{% for p in sorted_pages %}
      {% if p.layout == 'chapter' %}
        <li><a href="{{ site.baseurl }}{{ p.url }}">{{p.title}}</a><br />
        <em>{{ p.description }}</em>
        </li>
      {% endif %}
{% endfor %}
</ol>


<h3>Browser Requirements</h3>
<p>This tutorial has been extensively tested in Safari <code>v.11.1.2</code> onwards, but should perform as intended in any modern browser (Firefox, Chrome, Edge, etc.).
</p>

<h3>Acknowledgments</h3>

<p> We would like to thank the following people. The core WebPPL team and contributors for their ongoing work. Noah Goodman and Josh Tenenbaum for making web templates for executing WebPPL fragments freely available. </p>



</div>

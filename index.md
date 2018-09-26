---
layout: default
title: Probabilistic Models of Cognition - 2nd Edition
custom_js:
- assets/js/index.js
custom_css:
- assets/css/index.css
---

<div id="header">
  <h1 id='title'>Probabilistic Models of Cognition</h1>
  <hr class='edition' />
  <span class="authors">by Kenneth Nichols</span>
</div>

<br />
Voice and text based interaction have become the de facto standard for controlling internet connected appliances and devices in domestic settings cite:porcheron-etal2018. Despite their popularity, these systems are subject to a number of limitations, most notably, their inability to interpret the subtleties of meaning in everyday language use refp:Luger-etal:2016, refp:Moore:2017. This thesis aims to advance the current understanding of how utterances are interpreted in the context of built environmets, especially as they relate to vague and ambiguous commands. In particular, we argue that in order to make progress in this domain, it is necessary to develop a common framework for knitting together two core components of pragmatic language understanding: **contextual information** refp:Alegre-etal:2016, and **intutitive models** of how agents plan and carry out goals in environments. We demonstrate how these components can be used to audit the behavior of the system, enabling it to give reasoned explanations for why it behaved the way it did.

<div id='left'>


<h3>Accompanying Thesis</h3>
N. D. Goodman, J. B. Tenenbaum, and The ProbMods Contributors (2016). <i>Probabilistic Models of Cognition</i> (2nd ed.). Retrieved <span class="date">YYYY-MM-DD</span> from <code>https://probmods.org/</code><br /><a id="toggle-bibtex" href="#">[bibtex]</a>

<pre id="bibtex">
@misc{probmods2,
  title = {% raw %}{{Probabilistic Models of Cognition}}{% endraw %},
  edition = {Second},
  author = {Goodman, Noah D and Tenenbaum, Joshua B. and The ProbMods Contributors},
  year = {2016},
  howpublished = {\url{http://probmods.org/v2}},
  note = {Accessed: <span class="date"></span>}
}
</pre>

<h3>Local Installation Instructions</h3>

<p> All instructions are for Mac OS (or OSX) version <code>10.13.6</code> onwards. WebPPL is hosted locally using the <a href="https://nodejs.org/en/"> node.js</a>) runtime environment. The **node package manager** (npm) repository is used to download and install WebPPL. Running <code>npm install -g webppl</code> in a terminal installs the latest version of WebPPL from npm. Once installed, the webppl program <code>foo</code> can be run globally in the terminal using the <code>webppl</code> prefix, i.e. <code>webppl foo.webppl</code> </p>

<p>A number of WebPPL specific packages and external javascript libraries are required for running the provided code. In the root or home directory (<code>cd ~</code>), create a new folder (<code>mkdir .webppl</code>). From here, install the **webppl-viz** library using the following command: <code>npm install probmods/webppl-viz</code>. This requires the **Canvas** and **Cairo** graphics libraries. The former can be installed using npm: `npm install canvas`. Further installation instructions and a binary installer for Cairo can be found <a href="https://www.cairographics.org/download/"> here </a> </p>




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

<p> We would like to thank the following people. The core WebPPL team and contributors for their ongoing work. Noah Goodman and Josh Tenenbaum for making web templates for executing WebPPL fragments freely available. Greg Scontras for the web formatted </p>



</div>

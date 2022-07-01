---
title: Android API Levels
top: 101
categories:
  - Android
tags:
  - Android
  - SDK
description: Android各版本对应的SDK版本(持续更新)
comments: true
abbrlink: a7110e1c
date: 2021-10-07 17:26:26
---
<!--more-->
<meta name="referrer" content="no-referrer"/>

<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>API Levels | Android versions, SDK/API levels, version codes, codenames, and cumulative usage</title>
<meta name="description" content="A quick reference table of Android versions with SDK & API levels, version codes, codenames, cumulative usage, and more.">
<meta name="keywords" content="android,api,sdk,levels,version,codes,codename,usage,market,share,distribution">
<meta name="author" content="Eugene Belinski">
<link rel="stylesheet" href="/css/main.css">
<link rel="canonical" href="/">

<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">

<meta name="twitter:card" content="summary" />
<meta name="twitter:site" content="@EugeneBelinski" />
<meta name="twitter:creator" content="@EugeneBelinski" />
<meta property="og:type" content="website" />
<meta property="og:url" content="https://apilevels.com/" />
<meta property="og:title" content="Android API Levels" />
<meta property="og:description" content="A quick reference table of Android versions with SDK & API levels, version codes, codenames, cumulative usage, and more." />
<meta property="og:image" content="https://apilevels.com/logo.png" />
</head>
<body>
<div class="container">
<h1 id="site-title"><a href="/">Android API Levels</a></h1>
</div>
<div class="container"><div id="page-content">
<p>This is an overview of all Android versions and their corresponding identifiers for Android developers. Anyone is welcome to open <a href="https://github.com/ebelinski/apilevels">an issue or pull request</a>. Happy developing!</p>
<div id="compact-toc">
<ul id="markdown-toc">
<li><a href="#definitions" id="markdown-toc-definitions">Definitions</a> <ul>
<li><a href="#gradle-files" id="markdown-toc-gradle-files">Gradle files</a></li>
<li><a href="#code-files" id="markdown-toc-code-files">Code files</a></li>
</ul>
</li>
<li><a href="#footnotes" id="markdown-toc-footnotes">Footnotes</a></li>
<li><a href="#see-also" id="markdown-toc-see-also">See also</a></li>
</ul>
</div>
<div class="table-responsive">
<table class="full-width">
<tr>
<th>Version</th>
<th>SDK / API level</th>
<th><a href="https://developer.android.com/reference/kotlin/android/os/Build.VERSION_CODES">Version code</a></th>
<th>Codename</th>
<th>
Cumulative<br />usage
<sup id="fnref:1"><a href="#fn:1" class="footnote">1</a></sup>
</th>
<th>Year</th>
</tr>
<tr>
<td>
<b><a href="https://developer.android.com/about/versions/13">Android 13</a></b>
<sup class="beta">BETA</sup>
</td>
<td>Level 33</td>
<td><code>T</code></td>
<td>
Tiramisu
<sup id="fnref:2"><a href="#fn:2" class="footnote">2</a></sup>
</td>
<td rowspan="2"><i>No data</i></td>
<td rowspan="2"><i>TBA</i></td>
</tr>
<tr>
<td rowspan="3">
<b><a href="https://developer.android.com/about/versions/12">Android 12</a></b>
</td>
<td>
Level 32
<span class="subversion">Android 12L</span>
</td>
<td><code>S_V2</code></td>
<td rowspan="2">
Snow Cone
<sup id="fnref:2"><a href="#fn:2" class="footnote">2</a></sup>
</td>
</tr>
<tr>
<td>Level 31 <span class="subversion">Android 12</span></td>
<td><code>S</code></td>
<style>
  #cell146 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 14.6%, white 14.6%);
  }

  @media (prefers-color-scheme: dark) {
    #cell146 {
      background: linear-gradient(to right, #006236 14.6%, rgba(0, 0, 0, 0) 14.6%);
    }
  }
</style>
<td rowspan="2" id="cell146">
14.6%
</td>
<td rowspan="2">2021</td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li><code>targetSdk</code> <a href="https://developer.android.com/google/play/requirements/target-sdk">will need to be 31+</a> for new apps by August 2022 and updates by November 2022. <sup id="fnref:3"><a href="#fn:3" class="footnote">3</a></sup></li>
<li><code>targetSdk</code> <a href="https://support.google.com/googleplay/android-developer/answer/11926878">will need to be 31+</a> for <i>all existing apps</i> by November 2023. <sup id="fnref:4"><a href="#fn:4" class="footnote">4</a></sup></li>
</ul>
</td></tr>
<tr>
<td rowspan="2">
<b><a href="https://developer.android.com/about/versions/11">Android 11</a></b>
</td>
<td>Level 30</td>
<td><code>R</code></td>
<td>
Red Velvet Cake
<sup id="fnref:2"><a href="#fn:2" class="footnote">2</a></sup>
</td>
<style>
  #cell477 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 47.7%, white 47.7%);
  }

  @media (prefers-color-scheme: dark) {
    #cell477 {
      background: linear-gradient(to right, #006236 47.7%, rgba(0, 0, 0, 0) 47.7%);
    }
  }
</style>
<td rowspan="2" id="cell477">
47.7%
</td>
<td rowspan="2">2020</td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li><code>targetSdk</code> <a href="https://developer.android.com/google/play/requirements/target-sdk">must be 30+</a> for new apps and app updates. <sup id="fnref:3"><a href="#fn:3" class="footnote">3</a></sup></li>
<li><code>targetSdk</code> <a href="https://support.google.com/googleplay/android-developer/answer/11926878">will need to be 30+</a> for <i>all existing apps</i> by November 2022. <sup id="fnref:4"><a href="#fn:4" class="footnote">4</a></sup></li>
</ul>
</td></tr>
<tr>
<td>
<b><a href="https://developer.android.com/about/versions/10">Android 10</a></b>
</td>
<td>Level 29</td>
<td><code>Q</code></td>
<td>
Quince Tart
<sup id="fnref:2"><a href="#fn:2" class="footnote">2</a></sup>
</td>
<style>
  #cell703 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 70.3%, white 70.3%);
  }

  @media (prefers-color-scheme: dark) {
    #cell703 {
      background: linear-gradient(to right, #006236 70.3%, rgba(0, 0, 0, 0) 70.3%);
    }
  }
</style>
<td rowspan="1" id="cell703">
70.3%
</td>
<td>2019</td>
</tr>
<tr>
<td rowspan="2">
<b><a href="https://developer.android.com/about/versions/pie">Android 9</a></b>
</td>
<td>Level 28</td>
<td><code>P</code></td>
<td>Pie</td>
<style>
  #cell816 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 81.6%, white 81.6%);
  }

  @media (prefers-color-scheme: dark) {
    #cell816 {
      background: linear-gradient(to right, #006236 81.6%, rgba(0, 0, 0, 0) 81.6%);
    }
  }
</style>
<td rowspan="2" id="cell816">
81.6%
</td>
<td rowspan="2">2018</td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li><code>targetSdk</code> <a href="https://developer.android.com/google/play/requirements/target-sdk">must be 28+</a> for new Wear OS apps and Wear OS app updates.</li>
</ul>
</td></tr>
<tr>
<td rowspan="2">
<b><a href="https://developer.android.com/about/versions/oreo">Android 8</a></b>
</td>
<td>Level 27 <span class="subversion">Android 8.1</span></td>
<td><code>O_MR1</code></td>
<td rowspan="2">Oreo</td>
<style>
  #cell875 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 87.5%, white 87.5%);
  }

  @media (prefers-color-scheme: dark) {
    #cell875 {
      background: linear-gradient(to right, #006236 87.5%, rgba(0, 0, 0, 0) 87.5%);
    }
  }
</style>
<td rowspan="1" id="cell875">
87.5%
</td>
<td rowspan="2">2017</td>
</tr>
<tr>
<td>Level 26 <span class="subversion">Android 8.0</span></td>
<td><code>O</code></td>
<style>
  #cell902 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 90.2%, white 90.2%);
  }

  @media (prefers-color-scheme: dark) {
    #cell902 {
      background: linear-gradient(to right, #006236 90.2%, rgba(0, 0, 0, 0) 90.2%);
    }
  }
</style>
<td rowspan="1" id="cell902">
90.2%
</td>
</tr>
<tr>
<td rowspan="2">
<b><a href="https://developer.android.com/about/versions/nougat">Android 7</a></b>
</td>
<td>Level 25 <span class="subversion">Android 7.1</span></td>
<td><code>N_MR1</code></td>
<td rowspan="2">Nougat</td>
<style>
  #cell918 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 91.8%, white 91.8%);
  }

  @media (prefers-color-scheme: dark) {
    #cell918 {
      background: linear-gradient(to right, #006236 91.8%, rgba(0, 0, 0, 0) 91.8%);
    }
  }
</style>
<td rowspan="1" id="cell918">
91.8%
</td>
<td rowspan="2">2016</td>
</tr>
<tr>
<td>Level 24 <span class="subversion">Android 7.0</span></td>
<td><code>N</code></td>
<style>
  #cell946 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 94.6%, white 94.6%);
  }

  @media (prefers-color-scheme: dark) {
    #cell946 {
      background: linear-gradient(to right, #006236 94.6%, rgba(0, 0, 0, 0) 94.6%);
    }
  }
</style>
<td rowspan="1" id="cell946">
94.6%
</td>
</tr>
<tr>
<td>
<b><a href="https://developer.android.com/about/versions/marshmallow">Android 6</a></b>
</td>
<td>Level 23</td>
<td><code>M</code></td>
<td>Marshmallow</td>
<style>
  #cell971 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 97.1%, white 97.1%);
  }

  @media (prefers-color-scheme: dark) {
    #cell971 {
      background: linear-gradient(to right, #006236 97.1%, rgba(0, 0, 0, 0) 97.1%);
    }
  }
</style>
<td rowspan="1" id="cell971">
97.1%
</td>
<td>2015</td>
</tr>
<tr>
<td rowspan="3">
<b><a href="https://developer.android.com/about/versions/lollipop">Android 5</a></b>
</td>
<td>Level 22 <span class="subversion">Android 5.1</span></td>
<td><code>LOLLIPOP_MR1</code></td>
<td rowspan="2">Lollipop</td>
<style>
  #cell987 {
    width: 100%
    height: 100%;
    background: linear-gradient(to right, #60ECAD 98.7%, white 98.7%);
  }

  @media (prefers-color-scheme: dark) {
    #cell987 {
      background: linear-gradient(to right, #006236 98.7%, rgba(0, 0, 0, 0) 98.7%);
    }
  }
</style>
<td rowspan="1" id="cell987">
98.7%
</td>
<td>2015</td>
</tr>
<tr>
<td>Level 21 <span class="subversion">Android 5.0</span></td>
<td><code>LOLLIPOP</code>, <code>L</code></td>
<td rowspan="24"><i>No data</i></td>
<td rowspan="3">2014</td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li><a href="https://developer.android.com/jetpack/compose">Jetpack Compose</a> requires a <code>minSdk</code> of 21 or higher.</li>
</ul>
</td></tr>
<tr>
<td rowspan="9"><b>Android 4</b></td>
<td>
Level 20
<span class="subversion">Android 4.4W</span> <sup id="fnref:5"><a href="#fn:5" class="footnote">5</a></sup>
</td>
<td><code>KITKAT_WATCH</code></td>
<td rowspan="2">KitKat</td>
</tr>
<tr>
<td>
Level 19
<span class="subversion">Android 4.4</span>
</td>
<td><code>KITKAT</code></td>
<td rowspan="3">2013</td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li>Google Play services <a href="https://android-developers.googleblog.com/2021/07/google-play-services-discontinuing-jelly-bean.html">do not support Android versions</a> below API level 19.</li>
</ul>
</td></tr>
<tr>
<td>Level 18 <span class="subversion">Android 4.3</span></td>
<td><code>JELLYBEAN_MR2</code></td>
<td rowspan="3">Jelly Bean</td>
</tr>
<tr>
<td>Level 17 <span class="subversion">Android 4.2</span></td>
<td><code>JELLYBEAN_MR1</code></td>
 <td rowspan="2">2012</td>
</tr>
<tr>
<td>Level 16 <span class="subversion">Android 4.1</span></td>
<td><code>JELLYBEAN</code></td>
</tr>
<tr>
<td>Level 15 <span class="subversion">Android 4.0.3 – 4.0.4</span></td>
<td><code>ICE_CREAM_SANDWICH_MR1</code></td>
<td rowspan="2">Ice Cream Sandwich</td>
<td rowspan="7">2011</td>
</tr>
<tr>
<td>Level 14 <span class="subversion">Android 4.0.1 – 4.0.2</span></td>
<td><code>ICE_CREAM_SANDWICH</code></td>
</tr>
<tr class="table-notes"><td colspan="3">
<ul>
<li><a href="https://developer.android.com/jetpack">Jetpack</a>/<a href="https://developer.android.com/jetpack/androidx">AndroidX</a> libraries <a href="https://developer.android.com/topic/libraries/support-library#api-versions">require</a> a <code>minSdk</code> of 14 or higher.</li>
</ul>
</td></tr>
<tr>
<td rowspan="3"><b>Android 3</b></td>
<td>Level 13 <span class="subversion">Android 3.2</span></td>
<td><code>HONEYCOMB_MR2</code></td>
<td rowspan="3">Honeycomb</td>
</tr>
<tr>
<td>Level 12 <span class="subversion">Android 3.1</span></td>
<td><code>HONEYCOMB_MR1</code></td>
</tr>
<tr>
<td>Level 11 <span class="subversion">Android 3.0</span></td>
<td><code>HONEYCOMB</code></td>
</tr>
<tr>
<td rowspan="6"><b>Android 2</b></td>
<td>Level 10 <span class="subversion">Android 2.3.3 – 2.3.7</span></td>
<td><code>GINGERBREAD_MR1</code></td>
<td rowspan="2">Gingerbread</td>
</tr>
<tr>
<td>Level 9 <span class="subversion">Android 2.3.0 – 2.3.2</span></td>
<td><code>GINGERBREAD</code></td>
<td rowspan="3">2010</td>
</tr>
<tr>
<td>Level 8 <span class="subversion">Android 2.2</span></td>
<td><code>FROYO</code></td>
<td>Froyo</td>
</tr>
<tr>
<td>Level 7 <span class="subversion">Android 2.1</span></td>
<td><code>ECLAIR_MR1</code></td>
<td rowspan="3">Eclair</td>
</tr>
<tr>
<td>Level 6 <span class="subversion">Android 2.0.1</span></td>
<td><code>ECLAIR_0_1</code></td>
<td rowspan="5">2009</td>
</tr>
<tr>
<td>Level 5 <span class="subversion">Android 2.0</span></td>
<td><code>ECLAIR</code></td>
</tr>
<tr>
<td rowspan="4"><b>Android 1</b></td>
<td>Level 4 <span class="subversion">Android 1.6</span></td>
<td><code>DONUT</code></td>
<td>Donut</td>
</tr>
<tr>
<td>Level 3 <span class="subversion">Android 1.5</span></td>
<td><code>CUPCAKE</code></td>
<td>Cupcake</td>
</tr>
<tr>
<td>Level 2 <span class="subversion">Android 1.1</span></td>
<td><code>BASE_1_1</code></td>
<td>Petit Four</td>
</tr>
<tr>
<td>Level 1 <span class="subversion">Android 1.0</span></td>
<td><code>BASE</code></td>
<td><i>None</i></td>
<td>2008</td>
</tr>
</table>
</div>
<h2 id="definitions">Definitions</h2>
<h4 id="gradle-files">Gradle files</h4>
<div class="table-responsive">
<table class="full-width">
<tr>
<th class="nowrap">Kotlin variable</th>
<th class="nowrap">Groovy variable</th>
<th>Definition</th>
</tr>
<tr>
<td class="nowrap"><code>minSdk</code></td>
<td class="nowrap"><code>minSdkVersion</code></td>
<td>The minimum SDK version your app will support, defined in <code>build.gradle</code>. For example, if your <code>minSdk</code> is 26, this SDK version corresponse to API Level 26 and Android 8, so your app will only run on devices with Android 8 or higher.</td>
</tr>
<tr>
<td class="nowrap"><code>targetSdk</code></td>
<td class="nowrap"><code>targetSdkVersion</code></td>
<td>The SDK version that your app targets, defined in <code>build.gradle</code>. This should always be the same as <code>compileSdk</code>.</td>
</tr>
<tr>
<td class="nowrap"><code>compileSdk</code></td>
<td class="nowrap"><code>compileSdkVersion</code></td>
<td>The SDK version that your app compiles against, defined in <code>build.gradle</code>. Android Studio uses this SDK version to build your AABs and APKs. This should always be the same as <code>targetSdk</code>.</td>
</tr>
</table>
</div>
<h4 id="code-files">Code files</h4>
<div class="table-responsive">
<table class="full-width">
<tr>
<th>Variable</th>
<th>Definition</th>
</tr>
<tr>
<td class="nowrap"><code>Build.VERSION.SDK_INT</code></td>
<td>The SDK version of the Android OS currently running on the user's device. For example, on a device running Android 11, this value will be <code>30</code> (aka <code>Build.VERSION_CODES.R</code>), even if the target and compile SDK of the app is different.</td>
</tr>
</table>
</div>
<h2 id="footnotes">Footnotes</h2>
<div class="footnotes">
<ol>
<li id="fn:1">
<p>Cumulative usage distribution figures were last updated on <b>May 31, 2022</b> using data from <a href="https://gs.statcounter.com/android-version-market-share/mobile-tablet/worldwide">Statcounter GlobalStats</a>. These figures may have changed significantly since the last update. You may update the figures yourself with a <a href="https://github.com/ebelinski/apilevels">pull request</a>. <a href="#fnref:1" class="reversefootnote">↩</a></p>
</li>
<li id="fn:2">
<p>The codenames for Android 10 and above in the table are the internal codenames. Beginning with Android 10, Google dropped the usage of codenames publicly.</p>
</li>
<li id="fn:3">
<p>This requirement excludes new Wear OS apps and updates, which still only require a <code>targetSdk</code> <a href="https://developer.android.com/google/play/requirements/target-sdk">of 28 or higher</a>. <a href="#fnref:3" class="reversefootnote">↩</a></p>
</li>
<li id="fn:4">
<p>In 2022, Google began imposing <a href="https://support.google.com/googleplay/android-developer/answer/11926878">minimum <code>targetSdk</code> requirements for existing apps</a>, in addition new apps and app updates. Existing apps that are not updated to meet <code>targetSdk</code> requirements by their deadlines will be <a href="https://android-developers.googleblog.com/2022/04/expanding-plays-target-level-api-requirements-to-strengthen-user-security.html">subject to restrictions</a>. This requirement excludes Wear OS apps. <a href="#fnref:4" class="reversefootnote">↩</a></p>
</li>
<li id="fn:5">
<p>Android 4.4W is the first Android release for Android Wear. <a href="#fnref:5" class="reversefootnote">↩</a></p>
</li>
</ol>
</div>
<h2 id="see-also">See also</h2>
<ul>
<li><a href="https://developer.android.com/reference/android/os/Build.VERSION_CODES">Build.VERSION_CODES</a> official reference</li>
<li><a href="https://source.android.com/setup/start/build-numbers">Codenames, Tags, and Build Numbers</a></li>
</ul>
</div></div>
<div class="container">
<div id="site-footer">
<p>
<a href="https://github.com/ebelinski/apilevels">Source</a> |
Maintained by <a href="https://twitter.com/EugeneBelinski">Eugene 👨🏻‍💻</a> and <a href="https://github.com/ebelinski/apilevels/graphs/contributors">others</a>
</p>
</div>
</div>
<script async defer src="https://si.apilevels.com/latest.js"></script>
<noscript><img src="https://si.apilevels.com/noscript.gif" alt="" referrerpolicy="no-referrer-when-downgrade" /></noscript>
</body>
</html>

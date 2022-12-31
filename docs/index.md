<!DOCTYPE html>
<html>

<head>
  <title>README</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.3/dist/katex.min.css">


  <style>
    /**
 * prism.js Github theme based on GitHub's theme.
 * @author Sam Clarke
 */
    code[class*="language-"],
    pre[class*="language-"] {
      color: #333;
      background: none;
      font-family: Consolas, "Liberation Mono", Menlo, Courier, monospace;
      text-align: left;
      white-space: pre;
      word-spacing: normal;
      word-break: normal;
      word-wrap: normal;
      line-height: 1.4;

      -moz-tab-size: 8;
      -o-tab-size: 8;
      tab-size: 8;

      -webkit-hyphens: none;
      -moz-hyphens: none;
      -ms-hyphens: none;
      hyphens: none;
    }

    /* Code blocks */
    pre[class*="language-"] {
      padding: .8em;
      overflow: auto;
      /* border: 1px solid #ddd; */
      border-radius: 3px;
      /* background: #fff; */
      background: #f5f5f5;
    }

    /* Inline code */
    :not(pre)>code[class*="language-"] {
      padding: .1em;
      border-radius: .3em;
      white-space: normal;
      background: #f5f5f5;
    }

    .token.comment,
    .token.blockquote {
      color: #969896;
    }

    .token.cdata {
      color: #183691;
    }

    .token.doctype,
    .token.punctuation,
    .token.variable,
    .token.macro.property {
      color: #333;
    }

    .token.operator,
    .token.important,
    .token.keyword,
    .token.rule,
    .token.builtin {
      color: #a71d5d;
    }

    .token.string,
    .token.url,
    .token.regex,
    .token.attr-value {
      color: #183691;
    }

    .token.property,
    .token.number,
    .token.boolean,
    .token.entity,
    .token.atrule,
    .token.constant,
    .token.symbol,
    .token.command,
    .token.code {
      color: #0086b3;
    }

    .token.tag,
    .token.selector,
    .token.prolog {
      color: #63a35c;
    }

    .token.function,
    .token.namespace,
    .token.pseudo-element,
    .token.class,
    .token.class-name,
    .token.pseudo-class,
    .token.id,
    .token.url-reference .token.variable,
    .token.attr-name {
      color: #795da3;
    }

    .token.entity {
      cursor: help;
    }

    .token.title,
    .token.title .token.punctuation {
      font-weight: bold;
      color: #1d3e81;
    }

    .token.list {
      color: #ed6a43;
    }

    .token.inserted {
      background-color: #eaffea;
      color: #55a532;
    }

    .token.deleted {
      background-color: #ffecec;
      color: #bd2c00;
    }

    .token.bold {
      font-weight: bold;
    }

    .token.italic {
      font-style: italic;
    }


    /* JSON */
    .language-json .token.property {
      color: #183691;
    }

    .language-markup .token.tag .token.punctuation {
      color: #333;
    }

    /* CSS */
    code.language-css,
    .language-css .token.function {
      color: #0086b3;
    }

    /* YAML */
    .language-yaml .token.atrule {
      color: #63a35c;
    }

    code.language-yaml {
      color: #183691;
    }

    /* Ruby */
    .language-ruby .token.function {
      color: #333;
    }

    /* Markdown */
    .language-markdown .token.url {
      color: #795da3;
    }

    /* Makefile */
    .language-makefile .token.symbol {
      color: #795da3;
    }

    .language-makefile .token.variable {
      color: #183691;
    }

    .language-makefile .token.builtin {
      color: #0086b3;
    }

    /* Bash */
    .language-bash .token.keyword {
      color: #0086b3;
    }

    /* highlight */
    pre[data-line] {
      position: relative;
      padding: 1em 0 1em 3em;
    }

    pre[data-line] .line-highlight-wrapper {
      position: absolute;
      top: 0;
      left: 0;
      background-color: transparent;
      display: block;
      width: 100%;
    }

    pre[data-line] .line-highlight {
      position: absolute;
      left: 0;
      right: 0;
      padding: inherit 0;
      margin-top: 1em;
      background: hsla(24, 20%, 50%, .08);
      background: linear-gradient(to right, hsla(24, 20%, 50%, .1) 70%, hsla(24, 20%, 50%, 0));
      pointer-events: none;
      line-height: inherit;
      white-space: pre;
    }

    pre[data-line] .line-highlight:before,
    pre[data-line] .line-highlight[data-end]:after {
      content: attr(data-start);
      position: absolute;
      top: .4em;
      left: .6em;
      min-width: 1em;
      padding: 0 .5em;
      background-color: hsla(24, 20%, 50%, .4);
      color: hsl(24, 20%, 95%);
      font: bold 65%/1.5 sans-serif;
      text-align: center;
      vertical-align: .3em;
      border-radius: 999px;
      text-shadow: none;
      box-shadow: 0 1px white;
    }

    pre[data-line] .line-highlight[data-end]:after {
      content: attr(data-end);
      top: auto;
      bottom: .4em;
    }

    html body {
      font-family: "Helvetica Neue", Helvetica, "Segoe UI", Arial, freesans, sans-serif;
      font-size: 16px;
      line-height: 1.6;
      color: #333;
      background-color: #fff;
      overflow: initial;
      box-sizing: border-box;
      word-wrap: break-word
    }

    html body>:first-child {
      margin-top: 0
    }

    html body h1,
    html body h2,
    html body h3,
    html body h4,
    html body h5,
    html body h6 {
      line-height: 1.2;
      margin-top: 1em;
      margin-bottom: 16px;
      color: #000
    }

    html body h1 {
      font-size: 2.25em;
      font-weight: 300;
      padding-bottom: .3em
    }

    html body h2 {
      font-size: 1.75em;
      font-weight: 400;
      padding-bottom: .3em
    }

    html body h3 {
      font-size: 1.5em;
      font-weight: 500
    }

    html body h4 {
      font-size: 1.25em;
      font-weight: 600
    }

    html body h5 {
      font-size: 1.1em;
      font-weight: 600
    }

    html body h6 {
      font-size: 1em;
      font-weight: 600
    }

    html body h1,
    html body h2,
    html body h3,
    html body h4,
    html body h5 {
      font-weight: 600
    }

    html body h5 {
      font-size: 1em
    }

    html body h6 {
      color: #5c5c5c
    }

    html body strong {
      color: #000
    }

    html body del {
      color: #5c5c5c
    }

    html body a:not([href]) {
      color: inherit;
      text-decoration: none
    }

    html body a {
      color: #08c;
      text-decoration: none
    }

    html body a:hover {
      color: #00a3f5;
      text-decoration: none
    }

    html body img {
      max-width: 100%
    }

    html body>p {
      margin-top: 0;
      margin-bottom: 16px;
      word-wrap: break-word
    }

    html body>ul,
    html body>ol {
      margin-bottom: 16px
    }

    html body ul,
    html body ol {
      padding-left: 2em
    }

    html body ul.no-list,
    html body ol.no-list {
      padding: 0;
      list-style-type: none
    }

    html body ul ul,
    html body ul ol,
    html body ol ol,
    html body ol ul {
      margin-top: 0;
      margin-bottom: 0
    }

    html body li {
      margin-bottom: 0
    }

    html body li.task-list-item {
      list-style: none
    }

    html body li>p {
      margin-top: 0;
      margin-bottom: 0
    }

    html body .task-list-item-checkbox {
      margin: 0 .2em .25em -1.8em;
      vertical-align: middle
    }

    html body .task-list-item-checkbox:hover {
      cursor: pointer
    }

    html body blockquote {
      margin: 16px 0;
      font-size: inherit;
      padding: 0 15px;
      color: #5c5c5c;
      background-color: #f0f0f0;
      border-left: 4px solid #d6d6d6
    }

    html body blockquote>:first-child {
      margin-top: 0
    }

    html body blockquote>:last-child {
      margin-bottom: 0
    }

    html body hr {
      height: 4px;
      margin: 32px 0;
      background-color: #d6d6d6;
      border: 0 none
    }

    html body table {
      margin: 10px 0 15px 0;
      border-collapse: collapse;
      border-spacing: 0;
      display: block;
      width: 100%;
      overflow: auto;
      word-break: normal;
      word-break: keep-all
    }

    html body table th {
      font-weight: bold;
      color: #000
    }

    html body table td,
    html body table th {
      border: 1px solid #d6d6d6;
      padding: 6px 13px
    }

    html body dl {
      padding: 0
    }

    html body dl dt {
      padding: 0;
      margin-top: 16px;
      font-size: 1em;
      font-style: italic;
      font-weight: bold
    }

    html body dl dd {
      padding: 0 16px;
      margin-bottom: 16px
    }

    html body code {
      font-family: Menlo, Monaco, Consolas, 'Courier New', monospace;
      font-size: .85em !important;
      color: #000;
      background-color: #f0f0f0;
      border-radius: 3px;
      padding: .2em 0
    }

    html body code::before,
    html body code::after {
      letter-spacing: -0.2em;
      content: "\00a0"
    }

    html body pre>code {
      padding: 0;
      margin: 0;
      font-size: .85em !important;
      word-break: normal;
      white-space: pre;
      background: transparent;
      border: 0
    }

    html body .highlight {
      margin-bottom: 16px
    }

    html body .highlight pre,
    html body pre {
      padding: 1em;
      overflow: auto;
      font-size: .85em !important;
      line-height: 1.45;
      border: #d6d6d6;
      border-radius: 3px
    }

    html body .highlight pre {
      margin-bottom: 0;
      word-break: normal
    }

    html body pre code,
    html body pre tt {
      display: inline;
      max-width: initial;
      padding: 0;
      margin: 0;
      overflow: initial;
      line-height: inherit;
      word-wrap: normal;
      background-color: transparent;
      border: 0
    }

    html body pre code:before,
    html body pre tt:before,
    html body pre code:after,
    html body pre tt:after {
      content: normal
    }

    html body p,
    html body blockquote,
    html body ul,
    html body ol,
    html body dl,
    html body pre {
      margin-top: 0;
      margin-bottom: 16px
    }

    html body kbd {
      color: #000;
      border: 1px solid #d6d6d6;
      border-bottom: 2px solid #c7c7c7;
      padding: 2px 4px;
      background-color: #f0f0f0;
      border-radius: 3px
    }

    @media print {
      html body {
        background-color: #fff
      }

      html body h1,
      html body h2,
      html body h3,
      html body h4,
      html body h5,
      html body h6 {
        color: #000;
        page-break-after: avoid
      }

      html body blockquote {
        color: #5c5c5c
      }

      html body pre {
        page-break-inside: avoid
      }

      html body table {
        display: table
      }

      html body img {
        display: block;
        max-width: 100%;
        max-height: 100%
      }

      html body pre,
      html body code {
        word-wrap: break-word;
        white-space: pre
      }
    }

    .markdown-preview {
      width: 100%;
      height: 100%;
      box-sizing: border-box
    }

    .markdown-preview .pagebreak,
    .markdown-preview .newpage {
      page-break-before: always
    }

    .markdown-preview pre.line-numbers {
      position: relative;
      padding-left: 3.8em;
      counter-reset: linenumber
    }

    .markdown-preview pre.line-numbers>code {
      position: relative
    }

    .markdown-preview pre.line-numbers .line-numbers-rows {
      position: absolute;
      pointer-events: none;
      top: 1em;
      font-size: 100%;
      left: 0;
      width: 3em;
      letter-spacing: -1px;
      border-right: 1px solid #999;
      -webkit-user-select: none;
      -moz-user-select: none;
      -ms-user-select: none;
      user-select: none
    }

    .markdown-preview pre.line-numbers .line-numbers-rows>span {
      pointer-events: none;
      display: block;
      counter-increment: linenumber
    }

    .markdown-preview pre.line-numbers .line-numbers-rows>span:before {
      content: counter(linenumber);
      color: #999;
      display: block;
      padding-right: .8em;
      text-align: right
    }

    .markdown-preview .mathjax-exps .MathJax_Display {
      text-align: center !important
    }

    .markdown-preview:not([for="preview"]) .code-chunk .btn-group {
      display: none
    }

    .markdown-preview:not([for="preview"]) .code-chunk .status {
      display: none
    }

    .markdown-preview:not([for="preview"]) .code-chunk .output-div {
      margin-bottom: 16px
    }

    .scrollbar-style::-webkit-scrollbar {
      width: 8px
    }

    .scrollbar-style::-webkit-scrollbar-track {
      border-radius: 10px;
      background-color: transparent
    }

    .scrollbar-style::-webkit-scrollbar-thumb {
      border-radius: 5px;
      background-color: rgba(150, 150, 150, 0.66);
      border: 4px solid rgba(150, 150, 150, 0.66);
      background-clip: content-box
    }

    html body[for="html-export"]:not([data-presentation-mode]) {
      position: relative;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      margin: 0;
      padding: 0;
      overflow: auto
    }

    html body[for="html-export"]:not([data-presentation-mode]) .markdown-preview {
      position: relative;
      top: 0
    }

    @media screen and (min-width:914px) {
      html body[for="html-export"]:not([data-presentation-mode]) .markdown-preview {
        padding: 2em calc(50% - 457px + 2em)
      }
    }

    @media screen and (max-width:914px) {
      html body[for="html-export"]:not([data-presentation-mode]) .markdown-preview {
        padding: 2em
      }
    }

    @media screen and (max-width:450px) {
      html body[for="html-export"]:not([data-presentation-mode]) .markdown-preview {
        font-size: 14px !important;
        padding: 1em
      }
    }

    @media print {
      html body[for="html-export"]:not([data-presentation-mode]) #sidebar-toc-btn {
        display: none
      }
    }

    html body[for="html-export"]:not([data-presentation-mode]) #sidebar-toc-btn {
      position: fixed;
      bottom: 8px;
      left: 8px;
      font-size: 28px;
      cursor: pointer;
      color: inherit;
      z-index: 99;
      width: 32px;
      text-align: center;
      opacity: .4
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] #sidebar-toc-btn {
      opacity: 1
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc {
      position: fixed;
      top: 0;
      left: 0;
      width: 300px;
      height: 100%;
      padding: 32px 0 48px 0;
      font-size: 14px;
      box-shadow: 0 0 4px rgba(150, 150, 150, 0.33);
      box-sizing: border-box;
      overflow: auto;
      background-color: inherit
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar {
      width: 8px
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar-track {
      border-radius: 10px;
      background-color: transparent
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar-thumb {
      border-radius: 5px;
      background-color: rgba(150, 150, 150, 0.66);
      border: 4px solid rgba(150, 150, 150, 0.66);
      background-clip: content-box
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc a {
      text-decoration: none
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc ul {
      padding: 0 1.6em;
      margin-top: .8em
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc li {
      margin-bottom: .8em
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc ul {
      list-style-type: none
    }

    html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview {
      left: 300px;
      width: calc(100% - 300px);
      padding: 2em calc(50% - 457px - 150px);
      margin: 0;
      box-sizing: border-box
    }

    @media screen and (max-width:1274px) {
      html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview {
        padding: 2em
      }
    }

    @media screen and (max-width:450px) {
      html body[for="html-export"]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview {
        width: 100%
      }
    }

    html body[for="html-export"]:not([data-presentation-mode]):not([html-show-sidebar-toc]) .markdown-preview {
      left: 50%;
      transform: translateX(-50%)
    }

    html body[for="html-export"]:not([data-presentation-mode]):not([html-show-sidebar-toc]) .md-sidebar-toc {
      display: none
    }

    /* Please visit the URL below for more information: */
    /*   https://shd101wyy.github.io/markdown-preview-enhanced/#/customize-css */

  </style>
</head>

<body for="html-export">
  <div class="mume markdown-preview  ">
    <p><a href="https://github.com/jodisfields" target="blank"><img align="center" title="Hello, World!"
          src="https://readme-typing-svg.herokuapp.com/?center=true&amp;vcenter=true&amp;width=600&amp;height=75&amp;font=rubik&amp;size=30&amp;duration=3000&amp;color=000000&amp;lines=P&#xEB;rsh&#xEB;ndetje%2c+My+Name+Is+Jodis!;&#x12A5;&#x12CD;+&#x1230;&#x120B;&#x121D;+&#x1290;&#x12CD;%2c+My+Name+Is+Jodis!;&#x623;&#x647;&#x644;&#x627;%2c+My+Name+Is+Jodis!;&#x532;&#x561;&#x580;&#x565;&#x582;+&#x541;&#x565;&#x566;%2c+My+Name+Is+Jodis!;Salam%2c+My+Name+Is+Jodis!;&#x9B9;&#x9CD;&#x9AF;&#x9BE;&#x9B2;&#x9CB;%2c+My+Name+Is+Jodis!;&#x928;&#x92E;&#x938;&#x94D;&#x924;&#x947;%2c+My+Name+Is+Jodis!;Szia%2c+My+Name+Is+Jodis!;Ol&#xE1;%2c+My+Name+Is+Jodis!;&#xA38;&#xA24;+&#xA38;&#xA4D;&#xA30;&#xA40;+&#xA05;&#xA15;&#xA3E;&#xA32;%2c+My+Name+Is+Jodis!;Buna+Ziua%2c+My+Name+Is+Jodis!;&#x41F;&#x440;&#x438;&#x432;&#x435;&#x442;%2c+My+Name+Is+Jodis!;Talofa%2c+My+Name+Is+Jodis!;Hal&#xF2;%2c+My+Name+Is+Jodis!;&#x417;&#x434;&#x440;&#x430;&#x432;&#x43E;%2c+My+Name+Is+Jodis!;Mhoro%2c+My+Name+Is+Jodis!;Hola%2c+My+Name+Is+Jodis!;&#xC548;&#xB155;&#xD558;&#xC138;&#xC694;%2c+My+Name+Is+Jodis!;&#x3053;&#x3093;&#x306B;&#x3061;&#x306F;%2c+My+Name+Is+Jodis!;&#x417;&#x434;&#x440;&#x430;&#x432;&#x43E;%2c+My+Name+Is+Jodis!;&#x421;&#x430;&#x439;&#x43D;+&#x423;&#x443;%2c+My+Name+Is+Jodis!;&#x928;&#x92E;&#x938;&#x94D;&#x915;&#x93E;&#x930;%2c+My+Name+Is+Jodis!;Aloha%2c+My+Name+Is+Jodis!;Bonjour%2c+My+Name+Is+Jodis!;&#x5E9;&#x5DC;&#x5D5;&#x5DD;%2c+My+Name+Is+Jodis!;&#x3A7;&#x3B1;&#x3AF;&#x3C1;&#x3B5;&#x3C4;&#x3B5;%2c+My+Name+Is+Jodis!;Kamusta%2c+My+Name+Is+Jodis!;Hallo%2c+My+Name+Is+Jodis!;&#x4F60;&#x597D;%2c+My+Name+Is+Jodis!;&#x417;&#x434;&#x440;&#x430;&#x432;&#x435;&#x439;&#x442;&#x435;%2c+My+Name+Is+Jodis!;&#x414;&#x43E;&#x431;&#x440;&#x44B;+&#x414;&#x437;&#x435;&#x43D;&#x44C;%2c+My+Name+Is+Jodis!;Dia+Dhuit%2c+My+Name+Is+Jodis!;Ciao%2c+My+Name+Is+Jodis!;&#xEAA;&#xEB0;&#xE9A;&#xEB2;&#xE8D;&#xE94;&#xEB5;%2c+My+Name+Is+Jodis!;Kia+Ora%2c+My+Name+Is+Jodis!"
          alt="Hello, World!" height="100%" width="100%"></a></p>
    <h2 align="center">Software Engineer | Brisbane, Australia!</h2>
    <br>
    <div align="center">
      <a href="https://linkedin.com/in/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://kaggle.com/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/kaggle.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://stackoverflow.com/users/jodis-fields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/stack-overflow.svg"
          alt="jodis-fields" height="30" width="40"></a>
      <a href="https://www.codechef.com/users/jodisfields" target="blank"><img align="center"
          src="https://cdn.jsdelivr.net/npm/simple-icons@3.1.0/icons/codechef.svg" alt="jodisfields" height="30"
          width="40"></a>
      <a href="https://codesandbox.com/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/codesandbox.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://dribbble.com/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/dribbble.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://dev.to/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/devto.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://www.behance.net/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/behance.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://hashnode.com/@jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/hashnode.svg"
          alt="@jodisfields" height="30" width="40"></a>
      <a href="https://medium.com/@jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/medium.svg"
          alt="@jodisfields" height="30" width="40"></a>
      <a href="https://codepen.io/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/codepen.svg"
          alt="jodisfields" height="30" width="40"></a>
      <a href="https://www.hackerrank.com/jodisfields" target="blank"><img align="center"
          src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/hackerrank.svg"
          alt="jodisfields" height="30" width="40"></a><br><br>
    </div>
    <br>
    <div align="center">
      <a href="https://github.com/ryo-ma/github-profile-trophy" target="blank"> <img align="center"
          src="https://github-profile-trophy.vercel.app/?username=jodisfields&amp;margin-w=30&amp;no-frame=true&amp;no-bg=true&amp;rank=-?&amp;column=-1"
          alt="github-trophies"></a>
    </div>
    <br>
    <div align="center">
      <img align="center" src="https://github-readme-streak-stats.herokuapp.com/?user=jodisfields&amp;"
        alt="jodisfields" height width="32%">
      <img align="center"
        src="https://github-readme-stats.vercel.app/api?username=jodisfields&amp;show_icons=true&amp;locale=en"
        alt="jodisfields" height width="32%">
      <img align="center"
        src="https://spotify-github-profile.vercel.app/api/view?uid=jodis.fields1&amp;cover_image=true&amp;theme=novatorem"
        alt="jodisfields" height width="34%">
    </div>
    <h4 align="left">Languages and Tools:</h4>
    <br>
    <div>
      <a href="https://developer.android.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/android/android-original-wordmark.svg"
          alt="android" width="40" height="40"> </a> <a href="https://www.arduino.cc/" target="_blank" rel="noreferrer">
        <img src="https://cdn.worldvectorlogo.com/logos/arduino-1.svg" alt="arduino" width="40" height="40"> </a> <a
        href="https://aws.amazon.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/amazonwebservices/amazonwebservices-original-wordmark.svg"
          alt="aws" width="40" height="40"> </a> <a href="https://azure.microsoft.com/en-in/" target="_blank"
        rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/microsoft_azure/microsoft_azure-icon.svg"
          alt="azure" width="40" height="40"> </a> <a href="https://www.gnu.org/software/bash/" target="_blank"
        rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/gnu_bash/gnu_bash-icon.svg" alt="bash" width="40"
          height="40"> </a> <a href="https://circleci.com" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/circleci/circleci-icon.svg" alt="circleci" width="40" height="40"> </a>
      <a href="https://www.w3schools.com/css/" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/css3/css3-original-wordmark.svg"
          alt="css3" width="40" height="40"> </a> <a href="https://dart.dev" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/dartlang/dartlang-icon.svg" alt="dart" width="40" height="40"> </a> <a
        href="https://www.djangoproject.com/" target="_blank" rel="noreferrer"> <img
          src="https://cdn.worldvectorlogo.com/logos/django.svg" alt="django" width="40" height="40"> </a> <a
        href="https://www.docker.com/" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg"
          alt="docker" width="40" height="40"> </a> <a href="https://www.elastic.co" target="_blank" rel="noreferrer">
        <img src="https://www.vectorlogo.zone/logos/elastic/elastic-icon.svg" alt="elasticsearch" width="40"
          height="40">
      </a> <a href="https://expressjs.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/express/express-original-wordmark.svg"
          alt="express" width="40" height="40"> </a> <a href="https://www.figma.com/" target="_blank" rel="noreferrer">
        <img src="https://www.vectorlogo.zone/logos/figma/figma-icon.svg" alt="figma" width="40" height="40"> </a> <a
        href="https://firebase.google.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/firebase/firebase-icon.svg" alt="firebase" width="40" height="40"> </a>
      <a href="https://flask.palletsprojects.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/pocoo_flask/pocoo_flask-icon.svg" alt="flask" width="40" height="40">
      </a> <a href="https://flutter.dev" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/flutterio/flutterio-icon.svg" alt="flutter" width="40" height="40">
      </a>
      <a href="https://www.gatsbyjs.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/gatsbyjs/gatsbyjs-icon.svg" alt="gatsby" width="40" height="40"> </a>
      <a href="https://git-scm.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" alt="git" width="40" height="40"> </a> <a
        href="https://golang.org" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/go/go-original.svg" alt="go" width="40"
          height="40"> </a> <a href="https://grafana.com" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/grafana/grafana-icon.svg" alt="grafana" width="40" height="40"> </a> <a
        href="https://heroku.com" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/heroku/heroku-icon.svg" alt="heroku" width="40" height="40"> </a> <a
        href="https://www.w3.org/html/" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/html5/html5-original-wordmark.svg"
          alt="html5" width="40" height="40"> </a> <a href="https://gohugo.io/" target="_blank" rel="noreferrer"> <img
          src="https://api.iconify.design/logos-hugo.svg" alt="hugo" width="40" height="40"> </a> <a
        href="https://ifttt.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/ifttt/ifttt-ar21.svg" alt="ifttt" width="40" height="40"> </a> <a
        href="https://www.java.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/java/java-original.svg" alt="java"
          width="40" height="40"> </a> <a href="https://www.elastic.co/kibana" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/elasticco_kibana/elasticco_kibana-icon.svg" alt="kibana" width="40"
          height="40"> </a> <a href="https://kotlinlang.org" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/kotlinlang/kotlinlang-icon.svg" alt="kotlin" width="40" height="40">
      </a>
      <a href="https://kubernetes.io" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg" alt="kubernetes" width="40"
          height="40">
      </a> <a href="https://www.linux.org/" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/linux/linux-original.svg" alt="linux"
          width="40" height="40"> </a> <a href="https://mariadb.org/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/mariadb/mariadb-icon.svg" alt="mariadb" width="40" height="40"> </a> <a
        href="https://www.mysql.com/" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/mysql/mysql-original-wordmark.svg"
          alt="mysql" width="40" height="40"> </a> <a href="https://nextjs.org/" target="_blank" rel="noreferrer"> <img
          src="https://cdn.worldvectorlogo.com/logos/nextjs-2.svg" alt="nextjs" width="40" height="40"> </a> <a
        href="https://www.nginx.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/nginx/nginx-original.svg" alt="nginx"
          width="40" height="40"> </a> <a href="https://nim-lang.org/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/nim-lang/nim-lang-icon.svg" alt="nim" width="40" height="40"> </a> <a
        href="https://nodejs.org" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/nodejs/nodejs-original-wordmark.svg"
          alt="nodejs" width="40" height="40"> </a> <a href="https://pandas.pydata.org/" target="_blank"
        rel="noreferrer">
        <img
          src="https://raw.githubusercontent.com/devicons/devicon/2ae2a900d2f041da66e950e4d48052658d850630/icons/pandas/pandas-original.svg"
          alt="pandas" width="40" height="40"> </a> <a href="https://www.postgresql.org" target="_blank"
        rel="noreferrer">
        <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/postgresql/postgresql-original-wordmark.svg"
          alt="postgresql" width="40" height="40"> </a> <a href="https://postman.com" target="_blank" rel="noreferrer">
        <img src="https://www.vectorlogo.zone/logos/getpostman/getpostman-icon.svg" alt="postman" width="40"
          height="40">
      </a> <a href="https://www.python.org" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python"
          width="40" height="40"> </a> <a href="https://reactnative.dev/" target="_blank" rel="noreferrer"> <img
          src="https://reactnative.dev/img/header_logo.svg" alt="reactnative" width="40" height="40"> </a> <a
        href="https://www.rust-lang.org" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/rust/rust-plain.svg" alt="rust"
          width="40" height="40"> </a> <a href="https://sass-lang.com" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/devicons/devicon/master/icons/sass/sass-original.svg" alt="sass"
          width="40" height="40"> </a> <a href="https://scikit-learn.org/" target="_blank" rel="noreferrer"> <img
          src="https://upload.wikimedia.org/wikipedia/commons/0/05/Scikit_learn_logo_small.svg" alt="scikit_learn"
          width="40" height="40"> </a> <a href="https://www.selenium.dev" target="_blank" rel="noreferrer"> <img
          src="https://raw.githubusercontent.com/detain/svg-logos/780f25886640cef088af994181646db2f6b1a3f8/svg/selenium-logo.svg"
          alt="selenium" width="40" height="40"> </a> <a href="https://www.sqlite.org/" target="_blank"
        rel="noreferrer">
        <img src="https://www.vectorlogo.zone/logos/sqlite/sqlite-icon.svg" alt="sqlite" width="40" height="40"> </a> <a
        href="https://svelte.dev" target="_blank" rel="noreferrer"> <img
          src="https://upload.wikimedia.org/wikipedia/commons/1/1b/Svelte_Logo.svg" alt="svelte" width="40" height="40">
      </a> <a href="https://www.tensorflow.org" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/tensorflow/tensorflow-icon.svg" alt="tensorflow" width="40"
          height="40">
      </a> <a href="https://www.vagrantup.com/" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/vagrantup/vagrantup-icon.svg" alt="vagrant" width="40" height="40">
      </a>
      <a href="https://www.adobe.com/products/xd.html" target="_blank" rel="noreferrer"> <img
          src="https://cdn.worldvectorlogo.com/logos/adobe-xd.svg" alt="xd" width="40" height="40"> </a> <a
        href="https://zapier.com" target="_blank" rel="noreferrer"> <img
          src="https://www.vectorlogo.zone/logos/zapier/zapier-icon.svg" alt="zapier" width="40" height="40"> </a>
    </div>

  </div>











</body>

</html>


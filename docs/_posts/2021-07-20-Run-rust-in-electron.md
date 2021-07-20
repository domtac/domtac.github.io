---
layout: post
title:  "Run Rust in electron"
date:   2021-07-20 13:55:38 +0200
categories: rust webassembly electron
---

<div class="separator" style="clear: both; text-align: center;"><a href="https://1.bp.blogspot.com/-K3BxK5hXTiY/YPX4gfW1vII/AAAAAAAACYs/4MBWS2hWWc4ocaTzW33gdptdZI6IlU6MwCLcBGAsYHQ/s2048/wasm-ferris.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1290" data-original-width="2048" src="https://1.bp.blogspot.com/-K3BxK5hXTiY/YPX4gfW1vII/AAAAAAAACYs/4MBWS2hWWc4ocaTzW33gdptdZI6IlU6MwCLcBGAsYHQ/s320/wasm-ferris.png" width="320" /></a></div><br /><p><br /></div>

On tinkering with electron to get a grasp on that technology, my first attempt was how to move the logic part to rust and leave the presentation layer to electron. Using webassembly (wasm) was my first choice, so after a quick web search I stumbled across this [post](https://blog.logrocket.com/supercharge-your-electron-apps-with-rust/) by Anshul Goyal.&nbsp;</p><p>&nbsp;Though it helped me by pointing out the used basic technologies I still needed quite some time to get it running, partly because there has been a breaking change and partly because the blog post skipped some steps and was missing a github repo to look up the missing information. On getting it to run on my side I was reminded of the"How to draw an owl" meme (<a href="https://www.reddit.com/r/WebAssembly/comments/6tj8pl/how_can_i_get_wasm_to_run_in_electron_is_there_a/" rel="nofollow" target="_blank">reddit</a>):<br /></p><p><br /></p><div class="separator" style="clear: both; text-align: center;"><a href="https://external-preview.redd.it/DodWFQ9mQkVyWoKFa0ZIu12PYrPo3P2T0taaK-lgJCo.png?auto=webp&amp;s=c180684f48b01ff6f2cbc72e080067039943de07" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="453" data-original-width="530" src="https://external-preview.redd.it/DodWFQ9mQkVyWoKFa0ZIu12PYrPo3P2T0taaK-lgJCo.png?auto=webp&amp;s=c180684f48b01ff6f2cbc72e080067039943de07" /></a>&nbsp;</div><div class="separator" style="clear: both; text-align: justify;">So I decided to add a step-by-step guide. I will not go deep into details and focusing more on a reproducible guide to follow through.<br /></div><div class="separator" style="clear: both; text-align: justify;">&nbsp;</div><div class="separator" style="clear: both; text-align: justify;"><h2>Prerequisites <br /></h2></div><p>&nbsp;To follow on with the next steps you'll need to have <a href="https://www.rust-lang.org/tools/install" target="_blank">rust</a> and <a href="https://classic.yarnpkg.com/en/docs/install/#debian-stable" target="_blank">yarn</a> installed. This is how you check your versions plus the versions used on creating this post: </p><p>&nbsp;<br /></p>

```shell
$ cargo --version
cargo 1.53.0 (4369396ce 2021-04-27)
```
and
```shell
$ yarn --version
1.22.10
```


<div style="text-align: left;">As an editor I am using VSCode. <br /></div><div style="text-align: left;">&nbsp;</div><h2 style="text-align: left;">Setting up electron from boilerplate</h2><div style="text-align: left;">&nbsp;To get started, just clone the <a href="https://github.com/electron-react-boilerplate/electron-react-boilerplate" target="_blank">Electron boiler plate project</a> to your PC.</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><pre><code class="sh">$ git clone --depth 1 --single-branch https://github.com/electron-react-boilerplate/electron-react-boilerplate.git elec_wasm
$ cd elec_wasm
$ yarn</code></pre>
On starting the application by typing '<i>yarn start</i>' you should see something like this:</div><div class="separator" style="clear: both; text-align: center;"><a href="https://1.bp.blogspot.com/-o_4f3wjROQ8/YPXf-fBtfpI/AAAAAAAACYc/58Z8aDEdLywvouwmo-tJkDWLmDs0xbaBQCLcBGAsYHQ/s811/Hello%2BElectron%2BReact%2521_BoilerPlateWindow.bmp" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="534" data-original-width="811" height="297" src="https://1.bp.blogspot.com/-o_4f3wjROQ8/YPXf-fBtfpI/AAAAAAAACYc/58Z8aDEdLywvouwmo-tJkDWLmDs0xbaBQCLcBGAsYHQ/w450-h297/Hello%2BElectron%2BReact%2521_BoilerPlateWindow.bmp" width="450" /></a></div><br /><div style="text-align: left;">Hooray!</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><h2>Creating your WebAssembly</h2><div style="text-align: left;">In order to execute your rust code you'll need to pack it into Webassembly. Fortunately, <a href="https://rustwasm.github.io/docs/wasm-pack/" target="_blank">wasm-pack</a> will do this for us. </div><div style="text-align: left;">Following the installation guide:</div>
<pre><code>curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh</code></pre>  
  <div style="text-align: left;"><br /></div><div style="text-align: left;">You shall be able to set up a Hello world project, which will open an alert window on calling greet from your electron app.<br /></div></div><div style="text-align: left;">
<pre><code>wasm-pack new simple_webassembly
</code></pre>
For this demo, I just ran that command inside the src folder of the electron app.</div><div style="text-align: left;">Calling <pre><code class="sh">wasm-pack build</code></pre> in the <i>simple_webassembly</i> folder will generate your webassemlbly into the <i>pkg</i> folder.</div><div style="text-align: left;">&nbsp;</div><div style="text-align: left;"><h2 style="text-align: left;">Adding webassembly to your electron app

Before you can run wasm in your app you need to enable that. First you need to install it via yarn:

```sh
yarn add webassembly
yarn install
```

Secondly you need to add webassembly as an experimental feature to your webpack config as a [breaking change has been introduced between v4 and v5](https://webpack.js.org/migrate/5/#clean-up-configuration).

In order to activate this feature add this line:<

```js
experiments: {
    asyncWebAssembly: true
  },
```


to .erb/configs/webpack.config.base.js.    

Now we're set up to finally combine our app and our webassembly.    
To verify that your webassembly works, just paste following code into src/App.tsx:


```js
import React, { useEffect } from 'react';
import { BrowserRouter as Router, Switch, Route } from 'react-router-dom';
import icon from '../assets/icon.png';
import './App.global.css';


const Hello = () => {
  return (
    <div>
      <div className="Hello">
        <img width="400px" alt="icon" src={icon} />
      </div>
      <h1>electron-meets-rust-webassembly</h1>
    </div>
  );
};

export default function App() {
  useEffect(() => {
    import('./simple-webassembly/pkg/simple_webassembly').then((module) =>
    module.greet());
  }
  );
  return (
    <Router>
      <Switch>
        <Route path="/" component={Hello} />
      </Switch>
    </Router>
  );
}
```
{% highlight js %}
import React, { useEffect } from 'react';
import { BrowserRouter as Router, Switch, Route } from 'react-router-dom';
import icon from '../assets/icon.png';
import './App.global.css';


const Hello = () => {
  return (
    <div>
      <div className="Hello">
        <img width="400px" alt="icon" src={icon} />
      </div>
      <h1>electron-meets-rust-webassembly</h1>
    </div>
  );
};

export default function App() {
  useEffect(() => {
    import('./simple-webassembly/pkg/simple_webassembly').then((module) =>
    module.greet());
  }
  );
  return (
    <Router>
      <Switch>
        <Route path="/" component={Hello} />
      </Switch>
    </Router>
  );
}
{% endhighlight %}

One last time `yarn start` and enjoy the profit:

![](https://1.bp.blogspot.com/-fRwJkuzmInk/YPX3kX9NXnI/AAAAAAAACYk/imQeK8CUcbU-hfSN2FkAMFpA2mFLbwztwCLcBGAsYHQ/s818/ElectronWebassemblyFerris.bmp)

## Conclusion
Following the above mentioned steps you shall be ready and set to get your rust code running in your electron app. As easy as setting up the the webassembly using wasm-pack was, the process of loading it into electron was somehow counter-intuitive and required some tinkering.

As a next step I will try to get some performance insights comparing nativ code and webassembly. 
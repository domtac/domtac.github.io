---
layout: post
title:  "Run Rust in electron"
date:   2021-07-20 13:55:38 +0200
categories: rust webassembly electron
---

![](https://1.bp.blogspot.com/-K3BxK5hXTiY/YPX4gfW1vII/AAAAAAAACYs/4MBWS2hWWc4ocaTzW33gdptdZI6IlU6MwCLcBGAsYHQ/s2048/wasm-ferris.png){:height="70%" width="70%"}

## Quick start

If you want to start exploring rust and electron, just create a project from my [fork](https://github.com/domtac/electron-react-boilerplate) of the electron_boiler_plate project by clicking   
![]({{ site.url }}/assets/UseThisTemplate.png)



____

## Why

On tinkering with electron to get a grasp on that technology, my first attempt was how to move the logic part to rust. Using webassembly (wasm) was my first choice, so after a quick web search I stumbled across this [post](https://blog.logrocket.com/supercharge-your-electron-apps-with-rust/) by Anshul Goyal.    
Though it helped me by pointing out the used basic technologies I still needed quite some time to get it running, partly because there has been a breaking change and partly because the blog post skipped some steps and was missing a github repo to look up the missing information. On getting it to run on my side I was reminded of the "How to draw an owl" meme ([reddit](https://www.reddit.com/r/WebAssembly/comments/6tj8pl/how_can_i_get_wasm_to_run_in_electron_is_there_a/)):       
![](https://external-preview.redd.it/DodWFQ9mQkVyWoKFa0ZIu12PYrPo3P2T0taaK-lgJCo.png?auto=webp&amp;s=c180684f48b01ff6f2cbc72e080067039943de07)

So I decided to add a step-by-step guide. I will not go deep into details and focusing more on a reproducible guide to follow through.


## Prerequisites
To follow on with the next steps you'll need to have [rust](https://www.rust-lang.org/tools/install) and [yarn](https://classic.yarnpkg.com/en/docs/install/#debian-stable) installed. This is how you check your versions plus the versions used on creating this post:     

```shell
$ cargo --version
cargo 1.53.0 (4369396ce 2021-04-27)
```
and
```shell
$ yarn --version
1.22.10
```


As an editor I am using VSCode on Ubuntu. 

## Setting up electron from boilerplate
To get started, just clone the [Electron boiler plate project](https://github.com/electron-react-boilerplate/electron-react-boilerplate) to your PC.

```sh
$ git clone --depth 1 --single-branch https://github.com/electron-react-boilerplate/electron-react-boilerplate.git elec_wasm
$ cd elec_wasm
$ yarn
```

On starting the application by typing `yarn start` you should see something like this:
![](https://1.bp.blogspot.com/-o_4f3wjROQ8/YPXf-fBtfpI/AAAAAAAACYc/58Z8aDEdLywvouwmo-tJkDWLmDs0xbaBQCLcBGAsYHQ/s811/Hello%2BElectron%2BReact%2521_BoilerPlateWindow.bmp)

**Hooray!**    

## Creating your WebAssembly   
In order to execute your rust code you'll need to pack it into Webassembly. Fortunately, [wasm-pack](https://rustwasm.github.io/docs/wasm-pack/) will do this for us. 
Following the installation guide:
```sh
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```
will add wasm-pack tp your system.


You shall be able to set up a Hello world project, which will open an alert window on calling greet from your electron app.
```sh
wasm-pack new simple_webassembly
```

For this demo, I just ran that command inside the _src_ folder of the electron app.

Calling 
```sh
wasm-pack build
``` 
in the _simple_webassembly_ folder will generate your webassemlbly into the _pkg_ folder.

## Adding webassembly to your electron app

Before you can run wasm in your app you need to enable that. First you need to install it via yarn:

```sh
yarn add webassembly
yarn install
yarn upgrade
```

Secondly you need to add webassembly as an experimental feature to your webpack config as a [breaking change has been introduced between v4 and v5](https://webpack.js.org/migrate/5/#clean-up-configuration).

In order to activate this feature add this line:

```js
experiments: {
    asyncWebAssembly: true
  },
```
to .erb/configs/webpack.config.base.js.    

Now we're set up to finally combine our app and our webassembly.    
To verify that our webassembly works, just replace our code in src/App.tsx:


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

One last time `yarn start` and enjoy the profit:

![](https://1.bp.blogspot.com/-fRwJkuzmInk/YPX3kX9NXnI/AAAAAAAACYk/imQeK8CUcbU-hfSN2FkAMFpA2mFLbwztwCLcBGAsYHQ/s818/ElectronWebassemblyFerris.bmp)

## Conclusion
Following the above mentioned steps you shall be ready and set to get your rust code running in your electron app. As easy as setting up the the webassembly using wasm-pack was, the process of loading it into electron was somehow counter-intuitive and required some tinkering.

As a next step I will try to get some performance insights comparing native code and webassembly. 


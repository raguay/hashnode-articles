## Svelte and NW.js Gotchas

My favorite JavaScript framework is [Svelte](https://svelte.dev/). You can find interesting tutorials on how I've been using Svelte on [my website](https://www.customct.com/#/tutorials/index). I've just starting using [Hashnode.com](https://hashnode.com/@raguay/joinme
) for my blog on my website using a sub-domain: http://blog.customct.com. Since I use [NameCheap.com](https://namecheap.pxf.io/OXJGn) for my DNS, it was easy to setup.

Svelte makes a great framework for creating desktop applications as well. Many developers like to use [Electron](https://www.electronjs.org/) for creating web technology based desktop applications, but my favorite is [NW.js](http://nwjs.io). But, Svelte and NW.js sometimes come into conflict with each other. 

Most of the conflicts are due to the packager, [Rollup](https://rollupjs.org/guide/en/). Rollup does tree shaking on the code base in order to minimize the build. But, this sometimes breaks libraries. I've also had many issues with Svelte not working with non-ES6 modules (in other words, libraries built for using `require()`). I've developed my own methods, but I'm also looking for other to add comments on better ways to do it!

This article isn't a how do to it type, but a collection of tricks and techniques that I have found that works. Therefore, I'm assuming the reader is familiar with these two technologies.

## Libraries using `require()`

As I already said, libraries using the `require()` to load in nodejs doesn’t always work well with Rollup. I’m told there is a way to get them to work, but all of my efforts haven’t produced good results. Especially with libraries like child_process, fs, path, ansi_up just to name a few. Therefore, the best way to load these that I’ve found so far is to load them in the `index.html` and attach them to the `window` global. Like this:

```JavaScript
window.child = require('child_process’);
```

Or, use the `globalThis` variable. Since Svelte is a JavaScript transpiler (ie: It takes the svelte files that has HTML, CSS, and JavaScript and creates the needed JavaScript and CSS files), it uses variable renaming to obscure the workings and make the final source more compact. But, that doesn’t work for referencing global variables. Therefore, always refer to these globally created variables using the `window` or `globalThis` global variable. Svelte will not rename those variables.  Therefore, you would write:

```JavaScript
window.child.exec(‘ls -al’, (output, err) => { console.log(output);});
```

Rollup does have extensions that is suppose to make using Node.js special libraries okay, but I could never get them to work. Anyone else have any ideas?

## Non-Web based Libraries

One of the nice things about using Svelte to design applications is the ability to use the browser to see what it looks like while coding. But, if you use many of the Node.js libraries that wouldn’t work in the browser (fs, path, etc), then you can’t really use that development environment. NW.js does have access to the developer tools of Chrome. But, you can't use the many other Chrome extensions that can help in solving issues. The best way around that is to have all calls to functions that are not browser friendly in a separate module (just a normal JavaScript file that Svelte doesn’t compile), and have a mock version of that file for using in the browser. The mock version has every function, but the functions just return dummy information or do nothing at all. That way, you can debug the graphics and UI in the browser and then work with the other functions just inside of NW.js. I’ve been using this approach in designing my ScriptBar program:


![E36877B2-6A20-4526-9512-7F18073E5359.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1603959291909/8UH0ctxwT.png)

All my low level, Node.js only code is in a single file of functions that I call from the main Svelte program. When I want to use the browser to debug something, then I run the dev environment that uses the mock file. 

ScriptBar is in alpha stage right now. It takes output from scripts or Node-Red nodes and displays it’s information. It’s similar to BitBar, TextBar, and Infinity Desktop with many other features those don’t have. It will run BitBar and TextBar scripts. It is a part of my ScriptPad program.

## Non-Reactive Parameters from the Calling JavaScript File

Svelte application is launched from a `main.js` JavaScript file. You can pass parameters down to the Svelte application, but it is a one way trip. Once the Svelte app is running, even if you call a function on the top level that adjusts a parameter, it will not trickle back down. For example:

Main.js contains:

```JavaScript
import Main from ‘./Main.svelte';

var perf = {};
perf.height = 50;
perf.width = 100;

function adjust(height, width) {
  perf.height = height;
  perf.width = width;
  
  // More code for adjusting the window position and size.
}

 // Launch the Svelte program.
  const app = new Main({
    target: document.body,
    props: {
      prefs: perf,
      adjust: adjust
    }
  });
 
 
```

Main.svelte contains:

```html
<div style=“width: {perfs.width}; height: {perfs.height}”>
   <!— Some clever application code —>
</div>

<script>
  export let perfs;
  export let adjust;

  // More clever code...
</script>
```

I’m keeping the code for changing the window in the `main.js` file and the application itself in the `Main.svelte` file. The Svelte part changes the window using the `adjust` function passed down to it. But, the changes to `perf` in `main.js` will not show up in the `Main.svelte` program area. All JavaScript files marked with `.js` extension will not be tracted for reactivity (simply because the compiler does nothing with these files other than minimization). Therefore, you need to make those changes yourself inside of the Svelte program files and still pass it on to the top for the NW.js calls to modify the window.

I can get around this by putting all the NW.js calls inside the Svelte program area, but then running it in a normal browser is never possible. Tradeoffs to be considered, but this took me a while to wrap my head around. I had gotten too use to Svelte reactivity inside the application.

## Conclusion

These are just a few issues I’ve ran into so far. As I hit more, I’ll be adding to this list. For now, let me know what you think and if you have any better ideas on how to over come these issues. No one learns until we all share our thoughts together!

I also hope this might of interested you into exploring NW.js and Svelte development for yourself.
## Controlling your Computer from a Mobile Device

A little while ago, someone asked what can be done with a older tablet or phone. I mentioned that you could control your computer with one. I’ve been doing it for a while with my Huawei phone or my Kindle Fire using a local web server on my computer. The person, I don’t remember who or on what forum we were talking, asked me to share it. Well, it was embedded deeply in another project. But, I decided to extract it and make it available.

Well, it is done and available [on my GitHub account](https://github.com/raguay/SvelteScriptServer). Feel free to download, expand upon it, and just have fun. If you add something interesting or just neat, send a pull request. It’s current state is far from ideal. You have to change the code to configure the buttons right now, but it gives a good idea of what you can do.

Word of Caution: it doesn’t have great security right now. I will be looking into adding that soon. In the meantime, I recommend only using it in a trusted and secure local network.

## File Organization

At the top level, there are three things: the server directory, the UI directory, and a `maskfile.md` file for using [mask](https://github.com/jakedeichert/mask). Mask is a great tool for running scripts on your project. I use it all the time.

The server directory has all the code for running the local Express.js server. You should not need to edit this file unless you are adding new features to the backend (like security).

The UI directory contains most of the code. Here, you can edit the tiles displayed and the actions they perform. This code is written in [Svelte](https://svelte.dev/) for speed and small foot print. Svelte is a great framework for writing Single Page Applications (SPA). 

The `UI/src` directory contains all the files for creating the user interface. The  `UI/src/main.js` file contains all non-Svelte code. Every other file is written in Svelte and has the `.svelte` extension.

The `UI/src/main.js` file contains the structures you will want to change to add more buttons and change the theming.

## Adding/Removing Buttons

The heart of the program is the `scriptList` structure. This is a two dimensional array of objects that define a tile (the button you press to evoke a program or script). The row of tiles is an array of those objects. So, to add a new row or remove a row, add or remove an array. 

The tile object is of the following format:

```json
{
  name: "Open fman",
  description: "Open a copy of the fman program.",
  script: "/Applications/fman.app/Contents/SharedSupport/bin/fman",
  args: ['&'],
  color: "lightgreen",
  textColor: "darkgreen",
  height: "70px",
  width: "30%"
}
```

The `name` is the name of the tile, `description` describes what action the tile performs, `program` is the program or script to run, `args` is an array of command line arguments used to run the command, `color` is the background color for the tile, `textColor` is the color for the text in the tile, `height` is the height of the tile, and `width` is the width of the tile. When describing the height or width, you can use percentages or fixed numbers in pixel, rem, or any other CSS measurement standard. When specifying the height, don’t use percentage as that will be the percentage of the row that it is on.

When you run a program, it will use the environment that the server is launched. If you launch the server in the root environment, then your programs will have the privileges and environment variables of that environment. Therefore, be very careful! I only run it in my own login. But, if it automatically launches in a startup script, then it will have the environment of the startup script that doesn’t always match your normal environment. Alfred users run into this issue often.

The only other interesting structure is the `styles` structure that controls the look of the background of the web page. Only two things can be currently configured: backgroundColor and textColor. These are very self explanatory.

## Running the Server

While in the top directory where the `mask file.md` file is, run the following command:

```sh
mask LaunchServer
```

This only works if you have the mask taskrunner on your computer. This command will compile the UI code and launch the Express.js server. Now open a web browser to `http://localhost:3000` and you will see the main page.

In the upper right corner is a button labeled `Show QR Code`.  By clicking this button, a QR code will be displayed. Scan that code with your mobile device and open the given web address in your browser. You will see the same page without the `Show QR Code` button. You are now setup to launch programs on your computer from your phone or tablet!

By clicking the QR code on your computer, it will go away. Same with clicking the button again.

## Conclusion

Now that you know about this software, play with it and let me know how you use it. If you add something useful, send a pull request. Above all, have fun!

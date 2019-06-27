Note: This document is a copy of a [Blogspot article by Criag Code](https://craigc0de.blogspot.com/2018/07/phoenix-reasonreact-full-stack.html). This version contains corrections to the sample code as well as how to get it working with a more modern version of Ubuntu.


### Phoenix + ReasonReact - Full-Stack Functional Programming

As I started designing a new web application project, I had the wonderful opportunity to choose which technologies to use to power it. Like many other developers, I have seen the technical debt that is inherent to some of the [potential choices](https://www.youtube.com/watch?v=bzkRVzciAZg), so I decided to take a step back and objectively evaluate which technologies would work best for this specific project. Based on [an interview with Jesper Louis Anderson](https://notamonadtutorial.com/interview-with-jesper-louis-andersen-about-erlang-haskell-ocaml-go-idris-the-jvm-software-and-b0de06440fbd), I decided to consider [functional programming languages](https://en.wikipedia.org/wiki/Functional_programming) as a starting point since they are the most "agile" types of languages.  

For the backend web server, I wanted to use a functional programming language, but I wasn’t ready to give up all of the niceties of [Rails](https://rubyonrails.org/) (convention over configuration, generators, ActiveRecord). This led me to the [Phoenix Framework](https://phoenixframework.org/), which is basically Rails, but instead of running on Ruby, it runs on [Elixir](https://elixir-lang.org/) in the [Erlang RunTime System](https://www.erlang.org/). This means that I get all of the niceties of Rails with all the [power of the Erlang RunTime System and OTP](https://stackoverflow.com/questions/8426897/erlangs-99-9999999-nine-nines-reliability) (fast, scalable, reliable). The choice was easy.  

For the client, I wanted to use [static typing](https://www.typescriptlang.org/), [immutability](https://facebook.github.io/immutable-js/), and [composable components](https://reactjs.org/), but I didn’t want the complexities of handling a different dependency for each of these criteria. That’s when I found [ReasonML](https://reasonml.github.io/). ReasonML is basically a re-skinned syntax of [OCaml](https://ocaml.org/) to make it look more like JavaScript and less like a scary functional programming language. Out of the box, ReasonML provides static typing, immutability, and other useful functional programming features ([Variants are pretty great!](https://reasonml.github.io/docs/en/variant)). Also, since ReasonML is also developed by Facebook developers, [the project is very closely tied with React](https://reasonml.github.io/reason-react/). Good enough for me!  

Now that I chose the winning team of technologies to use, my next step was to "google" all of the technologies together to see if someone has already written a blog post on how to wire them all up together. Unfortunately, even though these two technologies (Phoenix and ReasonReact) are awesome, they are still fairly new and haven’t yet gotten an invitation to the "popular web technologies" club, so I was left to wiring up the two technologies together myself. To help you, humble reader, from also not finding any results on using these two technologies together, I have put together this post to show you how I managed to get the two working together quite nicely.  

**So if you are ready to harness the awesome power of the Phoenix Framework as your server and ReasonReact as your client, please read on.** (To get spoilers for this post, [follow along with the complete code here](https://github.com/ctbucha/phoenix-reasonreact-example)).  


## Setting up the Phoenix Project

For this simple web application, we are going to start by creating a simple Phoenix project. If you aren’t familiar with the Phoenix framework yet (don’t worry; I haven’t heard of it until I started this project, and now I am obsessed with it), it is basically a Ruby on Rails project except instead of using Ruby, it uses Elixir and runs on top of the powerful Erlang RunTime System. If you aren’t familiar with the Erlang RunTime System, all you need to know now is that it is very fast, scalable, and [reliable](https://stackoverflow.com/questions/8426897/erlangs-99-9999999-nine-nines-reliability).

To get started, first we need to install Erlang and Elixir on the machine. Usually, when installing language runtimes on my machine, I like to use a version manager so I can keep my machine clean of binaries and configuration files, and also so I can easily switch between versions of the language runtime in case either something is not compatible with the current version I have installed or I want to try out the new features of the newest version. My version manager of choice for Erlang and Elixir is [asdf](https://github.com/asdf-vm/asdf).

To install asdf, follow the [installation instructions on the asdf readme](https://github.com/asdf-vm/asdf#setup). On macOS, this will look something like the following:
```
     git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.5.0  
     echo -e '\n. $HOME/.asdf/asdf.sh' >> ~/.bash_profile  
     echo -e '\n. $HOME/.asdf/completions/asdf.bash' >> ~/.bash_profile  
```

Next, restart your shell so that the PATH changes take effect.

Now, install Erlang:

```     
    asdf plugin-add erlang  
    asdf install erlang 20.3.8  
    asdf global erlang 20.3.8  
```

and Elixir:
```
    asdf plugin-add elixir  
    asdf install elixir 1.6.6  
    asdf global elixir 1.6.6  
```
Now that we have Erlang and Elixir installed, we can install the Phoenix framework. [Follow the Phoenix set up instructions](https://hexdocs.pm/phoenix/installation.html). They should look something like this:

     mix install local.hex  
     mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez  

Next, we have to make sure we have node.js running on the machine. I prefer to use [nvm](https://github.com/creationix/nvm) as the node version manager. Follow the [nvm installation instructions](https://github.com/creationix/nvm#install-script) to install nvm. The instructions should look something like this:
```
     curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash  
     nvm install v8.11.3  
     nvm use v8.11.3  
```
Now we have everything installed, and we’re ready to start our first project!

## Creating the Phoenix Project

If you are familiar with Rails generators, then you will be right at home here. We can create the entire starting point of a Phoenix application with a single command:
```
     $ mix phx.new myapp --no-ecto  
     * creating myapp/config/config.exs  
     * creating myapp/config/dev.exs  
     * creating myapp/config/prod.exs  
     * creating myapp/config/prod.secret.exs  
     * creating myapp/config/test.exs  
     * creating myapp/lib/myapp/application.ex  
     * creating myapp/lib/myapp.ex  
     * creating myapp/lib/myapp_web/channels/user_socket.ex  
     * creating myapp/lib/myapp_web/views/error_helpers.ex  
     * creating myapp/lib/myapp_web/views/error_view.ex  
     * creating myapp/lib/myapp_web/endpoint.ex  
     * creating myapp/lib/myapp_web/router.ex  
     * creating myapp/lib/myapp_web.ex  
     * creating myapp/mix.exs  
     * creating myapp/README.md  
     * creating myapp/test/support/channel_case.ex  
     * creating myapp/test/support/conn_case.ex  
     * creating myapp/test/test_helper.exs  
     * creating myapp/test/myapp_web/views/error_view_test.exs  
     * creating myapp/lib/myapp_web/gettext.ex  
     * creating myapp/priv/gettext/en/LC_MESSAGES/errors.po  
     * creating myapp/priv/gettext/errors.pot  
     * creating myapp/lib/myapp_web/controllers/page_controller.ex  
     * creating myapp/lib/myapp_web/templates/layout/app.html.eex  
     * creating myapp/lib/myapp_web/templates/page/index.html.eex  
     * creating myapp/lib/myapp_web/views/layout_view.ex  
     * creating myapp/lib/myapp_web/views/page_view.ex  
     * creating myapp/test/myapp_web/controllers/page_controller_test.exs  
     * creating myapp/test/myapp_web/views/layout_view_test.exs  
     * creating myapp/test/myapp_web/views/page_view_test.exs  
     * creating myapp/.gitignore  
     * creating myapp/assets/brunch-config.js  
     * creating myapp/assets/css/app.css  
     * creating myapp/assets/css/phoenix.css  
     * creating myapp/assets/js/app.js  
     * creating myapp/assets/js/socket.js  
     * creating myapp/assets/package.json  
     * creating myapp/assets/static/robots.txt  
     * creating myapp/assets/static/images/phoenix.png  
     * creating myapp/assets/static/favicon.ico  
     Fetch and install dependencies? [Yn] Y  
     * running mix deps.get  
     * running mix deps.compile  
     * running cd assets && npm install && node node_modules/brunch/bin/brunch build  
     We are all set! Go into your application by running:  
       $ cd myapp  
     Start your Phoenix app with:  
       $ mix phx.server  
     You can also run your app inside IEx (Interactive Elixir) as:  
       $ iex -S mix phx.server  
```

This command will create a new Phoenix project called "myapp" in the "myapp" directory. The "--no-ecto" flag tells the Phoenix generator not to include Ecto. Ecto is basically the persistence layer of a Phoenix application, and it allows your app to communicate with a backend database (for Rails developers, think ActiveRecord). Since we are just creating a simple frontend application, we are purposely keeping the application simple by not spinning up a database. If you need a database for your application, you can omit the "--no-ecto" flag, set up a database, and continue to follow along with this tutorial. If you want to see more details about the "mix phx.new" command, check out the great [Phoenix Up and Running guide](https://hexdocs.pm/phoenix/up_and_running.html).

The `mix phx.new` command will also prompt you to fetch and install dependencies. That is fine.

To test out if the application is working correctly, run the following commands:
```
     cd myapp  
     mix phx.server  
```
Then, with your favorite web browser, browse to http://localhost:4000\. If everything worked correctly, then you should now see the default Phoenix application page. 

## Adding ReasonReact to the Phoenix Project

When choosing a Javascript development environment, many developers have come to the agreement that [static typing is very important](https://www.typescriptlang.org/), [immutable objects are very important](https://facebook.github.io/immutable-js/), and [composability of components is very important](https://reactjs.org/). Since I am lazy about dependency management, I would like to get all of these features automatically built into a single dependency. Enter [ReasonReact](https://reasonml.github.io/reason-react/).

ReasonReact is basically React, but written in ReasonML instead of Javascript. ReasonML is basically a syntax re-write of OCaml to make it look more like JavaScript and less like a scary functional programming language.

To get ReasonReact to run in the browser, however, we are going to need to compile the ReasonML code into JavaScript. That is where [Bucklescript](https://bucklescript.github.io/) comes in. To install the necessary ReasonReact dependencies in our project, run the following commands:
```
     cd assets  
     npm install --save-dev bs-platform # this will likely need take a couple of minutes to run the first time  
     npm install --save react react-dom reason-react  
```

Now that ReasonReact is installed into the project, let’s create a small ReasonReact component that will get rendered on the home page of the application. Add the following files to your project (borrowed from the [reason-react-example project](https://github.com/reasonml-community/reason-react-example/tree/master/src/simple)):

`myapp/assets/js/SimpleRoot.re`
```
     ReactDOMRe.renderToElementWithId(<Page message="Hello!" />, "index");  
```
`myapp/assets/js/Page.re`
```
     let component = ReasonReact.statelessComponent("Page");  
     let handleClick = (_event, _self) => Js.log("clicked!");  
     let make = (~message, _children) => {  
      ...component,  
      render: self =>  
       <div onClick=(self.handle(handleClick))>  
        (ReasonReact.string(message))  
       </div>,  
     };  
```

Then add the following bucklescript configuration file to the assets directory:  

`myapp/assets/bsconfig.json`
```
     {  
      "name": "myapp",  
      "bsc-flags": ["-bs-no-version-header", "-bs-super-errors"],  
      "reason": {"react-jsx": 2},  
      "bs-dependencies": ["reason-react"],  
      "sources": [  
       {  
        "dir": "js",  
        "subdirs": true  
       }  
      ],  
      "package-specs": {  
       "module": "commonjs",  
       "in-source": true  
      },  
      "suffix": ".bs.js",  
      "refmt": 3  
     }  
```

And add the following commands to "scripts" in the assets/package.json file:

```
      ...  
      modules: {  
       autoRequire: {  
```
      ...  
      modules: {  
       autoRequire: {  
`myapp/assets/package.json`
```
      ...  
      "scripts": {  
       "build": "bsb -make-world",  
       "start": "bsb -make-world -w",  
       "clean": "bsb -clean-world",  
       ...  
```
This will allow us to run the Bucklescript commands through npm. To make sure everything is installed and working correctly, "cd" into your "assets" directory and run the following command to build your ReasonReact component:  
```
     npm run build  
```

If everything is working correctly, Bucklescript will compile all of the ReasonML files (*.re) in the assets/js directory recursively into JavaScript commonjs files with the extension .bs.js.

Now we have the JavaScript files, but how do we get them to actually run in any of the web pages served through the Phoenix web app? To accomplish this, we only need to make a few more changes.

By default, a Phoenix web app uses [brunch](https://brunch.io/) as the JavaScript build system (instead of [webpack](https://webpack.js.org/), [gulp](https://gulpjs.com/), [grunt](https://gruntjs.com/), etc.), so we’ll stick with that. Open up the assets/brunch-config.js file and add the JavaScript file of the root React component generated by Bucklescript to the "modules/autoRequire" object. This will make brunch build the final JavaScript package to run the ReactComponent when it gets loaded into the browser. Otherwise, the React component code will be bundled with the final JavaScript file, but it won’t be executed when it gets loaded into the browser.

`myapp/assets/brunch-config.js`
```
      ...  
      modules: {  
       autoRequire: {  
        "js/app.js": ["js/app", "js/SimpleRoot.bs.js"]  
       }  
      },  
      ...  
```

Now, the last step of displaying the React component in a web page served by the Phoenix web app is to create a "div" tag with the "id" of "index" so that the React component will know where to attach. We will do this by editing the default template created with the Phoenix web app. Delete all of the contents in the lib/myapp_web/templates/page/index.html.eex file and replace them with the following:

`myapp/lib/myapp_web/templates/page/index.html.eex`
```
      <div id="index"></div>  
```

Save this file and start up the Phoenix server again with mix phx.server in the root directory of the project (if it’s not already running). Load up the page in your favorite web browser again (http://localhost:4000), and if everything worked correctly, you should now see the React component displayed beneath the default Phoenix header. If this worked, then you are now running a ReasonReact client from a Phoenix web application! Great job! 

## Hot-loading ReasonReact

So one thing you know as a developer is you are not going to write the code once, compile it, and be done forever (unless you are a perfect programmer who can foresee all future requirements of the system 🔮). You are going to be constantly changing the ReasonReact code and seeing how it affects your application. To reduce the time it takes to write the code, save the code, compile the code, restart the server, write the code, save the code, and so on, you’ve likely used hot-reloading. Hot-reloading is when your development environment watches your code for changes, and then automatically compiles and redeploys the changes to your development server and you can immediately see the impact of the code you are currently working on. Luckily, Phoenix does this automatically with both the Elixir code in the "lib" directory and client code in the "assets" directory when you run the "mix phx.server" command. E.g. if you change (and save) and Elixir file in lib/myapp_web/controllers or a JavaScript file in assets/js/, then the changes will automatically be updated in your running development server. Awesome, right? Well what about ReasonML code? Phoenix doesn’t watch or update this code automatically. Bummer.


However, since Phoenix is watching for changes in JavaScript files in assets/js/, and when we make changes to the ReasonML files and compile them into JavaScript with Bucklescript, then Phoenix *does* see the updated JavaScript files and automatically hot-reloads them into your running development server. So all we have to do is have Bucklescript watch for changes to our ReasonML files and automatically compile the new JavaScript files when it sees changes. Fortunately, we already added this command to our assets/package.json file as the "start" script. If you run "npm run start" from the "assets" directory, then Bucklescript starts in "watch" mode, so it will automatically compile any ReasonML changes into JavaScript, Phoenix will see the changes to the generated JavaScript files and automatically hot-reload them into the running development server. Great! 

So now we know we can have hot-reloading of all of our code during development (both Elixir server code and ReasonReact frontend code) with the following two commands:
```
     cd assets && npm run start  
     mix phx.server  
```

But how do we combine both of these commands into a single command we can run when we’re ready for a long and productive coding session? (Because, yes, running a single command is far more effective than running _two_ commands. I’m that lazy.) We can just run both of these two commands asynchronously inside of our own Mix task! (For Ruby developers, Mix is basically Rake for Elixir. For other developers, Mix is basically Make but more fun.)

Create the following file in the lib/mix/tasks directory: 
`myapp/lib/mix/tasks/myapp.server.ex`
```
     defmodule Mix.Tasks.Myapp.Server do  
      use Mix.Task  
      @shortdoc "For Development - watches changes in Phoenix and ReasonML code."  
      @moduledoc """  
       For Development - watches changes in Phoenix and ReasonML code.  
      """  
      def run(_args) do  
       tasks = [  
        Task.async(fn -> Mix.shell.cmd "cd assets && npm run start" end),  
        Task.async(fn -> Mix.Task.run "phx.server" end),  
       ]  
       Enum.map(tasks, &Task.await(&1, :infinity))  
      end  
     end  
```

Now we compile the task by running "mix compile" in the root directory of our project.
```
     mix compile  
```

And finally, we can start our entire server, with hot-reloading for both server and client code, with the single command in the root directory of the project:

```
     mix myapp.server  
```

## Conclusion

Throughout this post, we learned how to install Erlang, Elixir, node.js, Phoenix, and Bucklescript, how to serve a ReasonReact client from a Phoenix web application, and how to set up hot-reloading for all of the code in the project. We went through each of these steps pretty quickly, so if you are having trouble with any of the specific technologies in any of the steps, I strongly advise you to look through the official documentation of that technology (each of the technologies discussed in this article has pretty great documentation). If nothing else, I hope this post encourages you to check out my two latest favorite technologies (Phoenix/Elixir and ReasonReact) and gives you an idea of how to synthesize the two to make even greater, fully-functional web applications. ([Full code here](https://github.com/ctbucha/phoenix-reasonreact-example))

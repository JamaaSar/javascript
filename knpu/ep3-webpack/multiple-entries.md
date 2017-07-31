# Multiple Entries

Our simple site actually has two different pages. We have this main lift stuff page. We also have a login page which has just a little bit of its own Java scripts. Well, the message there, if you have a really long user name, we get an error message. The template for this page is App/resources/FOS User Bundle/security/login.html.twig. And you see that it's including assets/js/login.js. The tray now is a very traditional non-webpack file. And that kind of sucks. I mean, I'm still using the suffix in function, I'm still relying on jquery to be included globally. What I want is I want to be able to webpackify this file. This is a very important thing. In our application, I want you to think of every distinct page that has its own Java script as its own application. So, from a Java script perspective, we have a login application. And we also have a lift stuff application. These are two separate applications. And they deserve two separate final output webpack files. In other words,

(silence)

In other words, in addition to just having build/rep_login.js, I want to actually create a build version of this login file as well, so on the login page I can also point to a nice webpackified file. I want to unlock the power of webpack for all of my individual Java script applications. So first, let's remove the self-executing function, and then since we're relying on the dollar sign variable, we'll say const dollar sign equals require jquery. Perfect!

Of course, the trick is that inside our webpack.config.js file, we really only have the ability so far to dump one file. Webpack looks at our rep_log.js file and dumps web/build/rep_log.js. So how can we actually get this to also look at login.js and dump a second file? Well, the answer is that entry can actually take an array. So [inaudible 00:02:43] copy the original path and set entry to open curly, close curly. For the first entry I'm going to say rep_log, set to that path, and then I'll repeat that for a new entry called login that points at our source login.js file. Now, in order to get webpack to dump two separate files, down here for the filename, you need to replace this with a very special [name].js. And the name is referring to these keys that I created up here in entry, rep_log and login. These could have been anything. For consistency, I have them the same as my filename, but you can choose any key you want for the entry, and the significance of that is that it just becomes the final dumped filename.

And you guys know as soon as we make a change webpack.config.js itself, you need to go back to your watch script, control c, and restart that. And when we do it, it dumps that login.js file. Boom! There is is in our build directory. So now on our login.html.twig, let's change the script tag to build/login.js. And head over, refresh the lift page first, still works, then I'll go to /login and it works, too. We now have the power to create as many different distinct Java script applications as we want.

In fact, we already have a third one. Inside the web assets js directory, in addition to login and replog, we have one called layout.js. And this just activates jquery bootstrap tool tips everywhere across the site, which is how we'll get the tool tips up here in our header. So even the layout itself, you can think of as a separate Jaca script application that deserves its own entry.

So I'm going to leave the self-executing block function for now. Let's just skip straight to webpack.config.js, make a new entry called layout. Then, inside of our base layout, so app/resources/views/base.html.twig, I'm going to change this to point to build/layout.js and go back to my webpack script, control c, run that, it saw our layout, it dumped our layout.js. You can see it down there. So I'm going to refresh, everything still works. Now we still have a few problems, the first being that right now, both login and relog.js, they both contain their own copies of jquery. You can see the file sizes for these are huge, so we're requiring our user to download jquery multiple times, once in each of these entries and still in our layout. That's something we're going to fix later. But first, we have a more immediate problem. When we refactor layout.js to use proper require statements, we're in for surprise.

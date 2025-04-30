## Porting 60k line Java code base to C#, or a story about mistakes of the past

```note : because egor want to shutdown the website, so I move the blog text here```

### Preface

Somewhere in the drafts of this blog, I have an article, that explains all the flaws of Java 8 (and a bunch of other languages). It probably will never be released, cause it’s just a pile of boring (and sad) facts, but I will list some of them, to explain my actions, that I did.
So, Java:

- **Has really old syntax**, compared, to C# or any other more modern language.
- **Packing an app requires or packing extra 100mb JVM, or having users to install JVM.** For example, Steam doesn’t ship JVM with it, and just overall packing .exe on Linux is a big pain.
- **"Write once, run anywhere" is a lie**. Java won’t run on consoles, such as Nintendo Switch.

Might be enough already, but there is also the framework, that BurningKnight Java Edition works on – LibGDX. Overall, it’s a great library, but:

- **What’s up with gamepad support?** Hotplug works only with LWJGL3, but that crashes on MacOS, gamepad remappings for millions of existing gamepads? No, you gotta figure them all out yourself.
- **Audio API is super limited** Might seem minor, but all the juice, that comes from lowpass filter, writing notes to audio source on fly, and other, is just impossible.

So yeah. It’s all was piling on me for over a year now, and I’ve been dreaming of porting the whole game to C# for a while now. Burning Knight git repo has csharp branch, that is 7 months, but it didn’t really go far beyond rendering an animation of a mummy.
I’ve knew it was a huge project to port, and I was scared of it.

### When my patience ended

Near the end of February I had a sudden idea of a compiler, that takes Java code and converts into C#. I’ve searched everywhere I could, and everything, that I found, was so outdated, so I would have to rewrite it, to get it working. Might sound like a big project, but it’s not really, thankfully, C# syntax is really close to Java.

So after 5 days of development (and being busy with real life stuff), [byejava](https://github.com/egordorichev/byejava) became a thing ([here is the dev thread on twitter](https://twitter.com/egordorichev/status/1098457329262084096)). The code on github is a bit broken right now (it did not place [] after arguments, and anonymous classes are not a thing in C#). I’ve manually converted all the kotlin files, that I had in Burning Knight, to Java, and run the whole 60k code base through the compiler. Guess how much time it took for it to process it? **Just 2.5 seconds!!!**

I’ve been expecting it to take around 5 minutes or so. I wouldn’t call the compiler the smartest thing ever, because it doesn’t have a resolver, it will probably break if it finds any errors in the java source code, and it just renames all variables to UpperCamelCase, again, because it has no resolver to check for private fields.


### A tiny bit of tech details on how byejava works

*If you are familiar with making AST-based interpreters, you can just ignore this part, you know everything about it, that you can think of.*

- Byejava stars of by reading the whole file into a string, and then splitting it into [tokens](https://stackoverflow.com/questions/4448661/what-is-the-exact-definition-of-token). I’ve been surprised, how many of them Java has.
- Then parser takes all the tokens and starts building [AST tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree). It tries to validate the code, but I’m still pretty sure, that it’s error handling ~~is the best~~.
- After that, we recursively descend into AST, renaming all the variables to UpperCamelCase, replacing String with string, HashMap with Dictionary, etc.
- And only now, the AST is passed onto emitter, who just takes the AST, and emits it into a C# code (it passes around a StringBuilder).
- After that, finally, the code is saved to a file, and we move onto the next one if we have one.

Yes, it’s not all that complex, the hardest part for me, was to figure out all the syntax corner cases, and how to tell apart cast expressions (ex. (int) a) and grouping expressions (ex. (a)). I’ve used a dirty hack, that looks at the case of identifier in the brackets, and decides from that, whether it is a cast or a grouping expression.

### Of course, C# code won’t work by itself

Yeah. My codebase didn’t rely too much on LibGDX, but still, I’m to this day adapting the code to Monogame (no, Unity, no). Monogame is pretty close to LibGDX, the only big flaw of it for me is that you have to compile shaders on a windows machine (I had to make a dual boot laptop for that).

Another thing, that I would mention, is that the code… [It wasn’t pretty…](https://twitter.com/egordorichev/status/1093236626216701952)
I’ve thrown away a bunch of code (like a **bunch**), I’ve rethought some core aspects of my engine design. I’ve started using ECS model, and I’ve considered using a bunch of programming patterns.

### What now?

Of course, I still have a lot of work to do, such as get the game to the state, where it was before. I’ve discovered some really neat game design tricks, and I’ve been putting them into a good use last week.
I’m sorry to say it, but you all know, that this will delay the project even more. But I think, yall still can be pretty hyped for the C# port, just because:

- Burning Knight will be on consoles!
- Local coop is going to be a thing 100%
- Twitch integration is going to be a thing 100%
- Mod support is already working, and you can write them in C#!
- Maybe, if everything goes well, we will have online multiplayer!

So yeah. But we have a long way to go. Taking, that I’m a solo programmer with this project, it will take time. I’m trying my best to move forward as fast, as I can, but at the same time, writing code, that I’m proud of, that I’m sure will be moddable later on, and code, that I can rely on.

**You can expect Part 2 in a month or so**

Cheers, Egor.

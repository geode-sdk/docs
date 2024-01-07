# Chapter 4.1: How Do You The Mod

Given that you are reading this handbook, you probably have some big ideas for mods you want to make. Maybe you want to be the next Absolute and make an awesome mod menu. Maybe you're not satisfied by how long BetterEdit is taking and want to create your own editor mod. Maybe you have something completely original planned. Whatever it is, it is important to understand how exactly you could approach turning your idea into reality.

Now, of course, it is obligatory to note that **there is no right or wrong way to make a mod**. There's no absolute do-this-and-youll-succeed process that is the same and works for every mod. Some mods require more effort than others, depending on a practically infinite amount of factors. However, this Chapter attempts to divide modding in general into 5 steps. Some may look at these steps and conclude that this is so generic it's practically useless, but it's good to remind ourselves now and then what it is we are actually doing.

## Step 1: Concretize Your Idea

**The first step is to really concretize your idea.** Often times, the ideas for the mods you want to make can be quite grandiose, like "I want to make GD more optimized" or "I want to make the editor better". Unfortunately, figuring out where to start actually working on the mod can prove to be a problem with large-scale ideas. In cases like these, you should try to **break the problem down into smaller parts**, like "I want to fix the scale controls in the editor being unable to rescale multiple objects at once" or "I want to add a 'Select by Group ID' button". Break down your idea into smaller parts, until you have something tangible you can actually start approaching.

## Step 2: Reverse Engineer

After figuring out what it is that you are actually going for, it's time to look into if it is even possible. For that, you will need to reverse engineer the game. If you want to fix some bugs in GD, first try to figure out what causes the bug. Is there a consistent setup or method to causing it? What classes and functions are related to this? Does it seem like a rounding error or something more severe?

## Step 3: Write Some Code

Once you have an idea on what might be a route to achieving what you want, write some code to try it out. If you don't know the exact magic words to write, refer to documentation and [ask around for help](https://discord.gg/AWWCUUfeA7).

## Step 4: Test

Once you have some code that compiles, the next step is to simply try it out. Load your mod into the game and see if it works. Send it to your friends to verify it works for them too. Make sure that it works all of the time, and that it doesn't have any unexpected side effects, like the game crashing after a while due to a memory leak.

## Step 5: Repeat ad nauseam

If your code doesn't work, reverse engineer a bit more to find out why, write some new code, and test that. Once you finish one feature, you may find yourself coming up with ideas for new ones, and so you can reverse engineer for those, write some code and test. Repeat basically forever until you have a finished product, or you get tired and start the next hobby project.

---

These may seem like completely dull and uninteresting steps for some, and perhaps like a glorified [WikiHow](https://www.youtube.com/watch?v=PSKQ3ZNQ_O8) article, but the point of that is to illustrate that there really is no specific process for making a mod. For example, reverse engineering may involve using Ghidra and IDA to analyze functions, or it may involve just looking around with DevTools. The code you write may be some in-depth template-filled C++ behemoth, or it may just be a few quick lines of test code. Testing may involve a larger group or just be you realizing you forgot to call `retain` on an array. Modding isn't an exact science; it's an art.

Even then, there are many important skills and tools every modder should get acquainted with, and in the next chapter we will look at one of the most important ones: [Reverse Engineering](/handbook/vol2/chap2_2.md)

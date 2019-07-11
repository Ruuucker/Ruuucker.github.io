**Update 8/19/17:** Scripts have been removed from this post. Instead they are available at [this repo](https://github.com/rucker/multi-firefox-mac).

# Running Multiple Firefox Profiles on MacOS
I have a use-case where it makes sense to run two Firefox profiles concurrently on MacOS. Furthermore, I want to be able to launch each of them via Spotlight rather than running a script by hand.

Some cursory research on a solution was slightly disappointing: articles I found appeared to be either [cumbersome and possibly outdated](https://spf13.com/post/managing-multiple-firefox-profiles-in-os-x/) or [not particularly helpful](http://kb.mozillazine.org/Opening_a_new_instance_of_Firefox_with_another_profile) due to the way MacOS app launches work. The former relied on mucking with a plist file and the application icon; the latter is not a complete solution because on MacOS, we're dealing with a convention where the `-no-remote` argument isn't easily written into a .desktop file as on Linux, or an application shortcut as on Windows. 

The following is my solution to this problem.

I have two complete Firefox application directories -- each containing a unique profile -- living at:  
/Applications/Firefox.app/Contents/MacOS  
~/Applications/Firefox (Other).app/Contents/MacOS  
(the "Other" profile began as a simple copy of the original Firefox.app directory)

## Setup
- Copy your existing Firefox.app directory and give it a name that suits you (see my example above). Later on, you can always create a new profile in the second Firefox directory using the [Profile Manager](http://kb.mozillazine.org/Profile_Manager)
- **Edit 8/19/17:** Manual setup instructions removed. Instead, clone [this repository](https://github.com/rucker/multi-firefox-mac) and run `fix-the-fox.sh` to do some further setup. This will replace each `firefox` binary with a small shell script that will run Firefox with `--no-remote` and the appropriate profile name.

What happens when the application is launched through Spotlight is `firefox` (the shell script) is invoked. It then calls the actual firefox binary, `firefox-bin`, with the absolute directory path prepended.

Because each profile is contained in its own Firefox application directory, Spotlight sees them as separate applications that can be launched freely at any time:

![]({{ site.url }}/assets/multi-firefox.png)

## Firefox Updates
One caveat to bear in mind is that each time Firefox receives an update, your `firefox` script will be replaced by the new version of the Firefox binary. This means a little maintenance is required after each update. `fix-the-fox.sh` will handle that for you.

Happy browsing!

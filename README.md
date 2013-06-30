bashtag-save
============

Introduction
------------

Inspired by [this](http://vignesh.foamsnet.com/2013/06/using-hash-tags-to-organize-bash-history.html) (HackerNews discussion [here](https://news.ycombinator.com/item?id=5965109)).  The idea is to use bash comments on the command line similarly to Twitter hashtags to leave strings in your history that you can find easily with ctrl+r.  We'll affectionately refer to these as "bashtags".  Here's an example:

    jpk@truth:~$ avconv -f alsa -ac 1 -i hw:1,0 -f x11grab -s 356x464 -i :0.0+121,443 -acodec libvo_aacenc -vcodec libx264 -preset fast -qp 0 -r 30 -async 30 video.mp4 #longassffmpegcommand

We just stuff the bashtag "#longassffmpegcommand" at the end of the command line so when we `ctrl+r, 'longass'` reverse search finds your command:

    (reverse-i-search)`longass': avconv -f alsa -ac 1 -i hw:1,0 -f x11grab -s 356x464 -i :0.0+121,443 -acodec libvo_aacenc -vcodec libx264 -preset fast -qp 0 -r 30 -async 30 video.mp4 #longassffmpegcommand

This is a nifty idea, but it begs the question: If you want to call up your huge command regularly, why not just put it in a script?  Good question.  That's what this joint is for.

Using bashtag-save
------------------

Adding the `bashtag-save` function to `$PROMPT_COMMAND` makes your shell aware of commands you type that have been tagged with tags starting with "#save:".  Like so:

    jpk@truth:~$ avconv -f alsa -ac 1 -i hw:1,0 -f x11grab -s 356x464 -i :0.0+121,443 -acodec libvo_aacenc -vcodec libx264 -preset fast -qp 0 -r 30 -async 30 video.mp4 #save:longassffmpegcommand
    (avconv does stuff...)
    jpk@truth:~$ ls -l ./scripts/saved/
    total 4
    -rwxrwxr-x 1 jpk jpk 198 Jun 30 15:57 longassffmpegcommand.sh
    jpk@truth:~$ cat ./scripts/saved/longassffmpegcommand.sh 
    #!/usr/bin/env bash
    avconv -f alsa -ac 1 -i hw:1,0 -f x11grab -s 356x464 -i :0.0+121,443 -acodec libvo_aacenc -vcodec libx264 -preset fast -qp 0 -r 30 -async 30 video.mp4 #save:longassffmpegcommand
    jpk@truth:~$ ./scripts/saved/longassffmpegcommand.sh
    (avconv does stuff again...)
    jpk@truth:~$ 

So tagging something with `#save:scriptname` will save the command to a script at at `~/scripts/saved/scriptname.sh`. You can leave it there for later, go edit it to build a more complex script around the command, whatever.

Installation
------------

First, clone this repo or download and save `bashtag-save.sh` somewhere.
Next, you should see something like this in your `~/.bashrc`:

    case "$TERM" in
    xterm*|rxvt*)
        PROMPT_COMMAND='echo -ne "\033]0;${USER}@${HOSTNAME}: ${PWD/$HOME/~}\007"'
        ;;
    *)
        ;;
    esac

You'll want to source `bashtag-save.sh` from wherever you saved it, then add the `bashtag-save` function to `PROMPT_COMMAND`. You probably don't want to lose whatever's already there, though.  So make another function nearby that will do whatever you already have in `PROMPT_COMMAND`, but also call `bashtag-save`.  You should end up with something like this:

    source path/to/bashtag-save.sh
    function prompt_commands {
        echo -ne "\033]0;${USER}@${HOSTNAME}: ${PWD/$HOME/~}\007"
        bashtag-save
    }
    case "$TERM" in
    xterm*|rxvt*)
        PROMPT_COMMAND='prompt_commands'
        ;;
    *)
        ;;
    esac

Last, you'll want to create the directory where the scripts are saved.  The default is `~/scripts/saved/`.

Customization
-------------

The `bashtag-save` function is really simple.  Open `bashtag-save.sh` and give it a look. To change the tag prefix the function uses to recognize if a command should be saved or not, just change the `save_tag_prefix` variable.  To change the path were scripts are saved, change the `script_path` variable.  To change what gets written to the script, change the stuff between the `EOL`s on the line that starts with `cat > $script_file`.


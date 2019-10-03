---
title: Extending DateTime in Ruby on Rails
layout: post
emoji: 💎
published: true
---

One way we're trying to bring consistency across all of our brands and apps is with
copy editing and writing style. Thankfully, it's not up to me to determine which
writing style to use.

It did, however, fall to me to figure out how we were going to tell `strftime` to format
the ante meridiem and post meridiem in our code, since there's no built-in way tell it to
return "a.m." or "p.m." (with the periods). The two possible ways
[offered to us
by `strftime`](http://man7.org/linux/man-pages/man3/strftime.3.html){:target="_blank"}{:rel="noopener"}
are `%P` and `%p` which return "am" and "PM" respectively.

```rb
# e.g. Nov. 20, 5:45pm
date.strftime("%b. %-d, %-l:%M %P")

# e.g. Sat - 6:00PM
date.strftime("%a - %-l:%M %p")
```

For some illogical reason, the `%P` returns in a lowercase format ("pm") while the
lowercase `%p` returns an uppercase format ("PM").

`¯\_(ツ)_/¯`

I dug around a bit and found a quick-and-dirty way to replace "am" as "a.m." but it
left a lot of room for error and wasn't very DRY, since I needed to use it in
multiple methods and across a few decorators.

```rb
date.strftime("%b. %-d, %-l:%M %P").sub("am", "a.m.").sub("pm", "p.m.")
```

So I enlisted the help of a buddy
[Richard](https://gitlab.com/d3d1rty){:target="_blank"}{:rel="noopener"}
who helped me look into the possibility of cleaning this up a bit. He suggested extending
the default `DateTime` class, which would make a nice reusable solution.

After a few unsuccessful attempts and a bit more research, we
[discovered](https://stackoverflow.com/a/5847393){:target="_blank"}{:rel="noopener"}
that you can extend a core class by putting it in the `config/initializers` directory,
since Rails will include everything in that directory by default.

Once we got that working, the next task was to correctly format regardless whether the
time being passed through was using `%p` or `%P`. A few minutes of Regex tomfoolery
later, we ended up with this:

```rb
# config/initializers/extensions/date_time.rb

class DateTime
  def am_pm_format(strftime_args)
    strftime(strftime_args).gsub(/am/i, "a.m.").gsub(/pm/i, "p.m.")
  end
end
```

Now we can pass `%p` or `%P` through and get the expected result in a reusable way:

```rb
# November 20 at 5:45 p.m.
date.am_pm_format("%B %-d at %-l:%M %P")
```

I'm still not 100% satisfied with those two `gsub` calls on there, but in the interest
of following *Make it Work, Make it Right, Make it Fast*, I'm pretty pleased with the
result.

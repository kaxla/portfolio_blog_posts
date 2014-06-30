###How to Create a Markdown-Friendly Blog in a Rails App

While making my new portfolio I wanted to include a simple blog. My requirements were that I be able to write my posts in markdown and that I be able to put the 3 most recent posts on the home page. Simple, right? Nope. This took me forever to figure out. 

At first I was just going to have a blog model/view/controller that I managed through [RailsAdmin](https://github.com/sferik/rails_admin) but I quickly realized that that was just going to give me a huge block of text with no formatting whatsoever which wasn't going to work in a very code-heavy blog.

Then I thought I could integrate a [Jekyll](http://jekyllrb.com/) blog into my existing app. There is a gem for this called [bloggy](https://github.com/zbruhnke/bloggy) that works really well but I ran into a problem when I wanted to display posts on the front page. Jekyll uses [Liquid](http://liquidmarkup.org/) as a templating language and I was using Rails' out-of-the-box ERB. As far as I can tell there is no way to make those two play nicely together. 

If I were starting from scratch I might have tried switching my entire app over to Liquid because it's pretty neat. It's extracted from Shopify and its primary use is for situations where you want to allow users control over their layout in a Rails app but don't want to let them run amok with Ruby on your server. Making things user-editable like this has become an interest of mine while I've been doing some freelance work for a friend who does custom Squarespace sites (yes, there is a huge market for this even though Squarespace is designed so that "anyone" can use it). But in this case I already had all my views laid out in ERB and it would have taken a long time to switch, so I eliminated that option.

Then I thought that maybe I could find something similar to Jekyll that uses ERB. There are several options: [Middleman](http://middlemanapp.com/), [Frank](https://github.com/blahed/frank), [Nanoc](http://nanoc.ws/), etc, but none of them could be easily integrated into an existing rails app the way Jekyll could with bloggy.

Finally, I stepped back and realized that there was probably a way to go with my original tactic and parse my markdown blog posts into HTML. That's when I found [Redcarpet](https://github.com/vmg/redcarpet), which was developed at Github and does exactly what I needed. 

I started off with [this Railscast](http://railscasts.com/episodes/272-markdown-with-redcarpet?view=asciicast) on it, but it's pretty outdated so I had to consult the docs to get things working correctly. Here's my final solution:

I added two gems to my gemfile:

```ruby
gem 'redcarpet'
gem 'coderay'
```

More on Coderay in a minute.

Next, I created a helper method for Redcarpet in app/helpers/application_helper.rb:

```ruby
 markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML,
    no_intra_emphasis: true, 
    fenced_code_blocks: true,   
    disable_indented_code_blocks: true)
return markdown.render(text).html_safe
end
```

Now in my view I can do this: 

```ruby
<%= markdown(@blog.body) %> 
```

and it will convert my markdown into HTML automagically. 

That works fine if you're just writing a blog about your cat or something, but if you are writing a blog about Rails development you're going to have a lot of code. If you don't care about syntax highlighting and just want your code to be differentiated from your explanations somehow you could just fiddle with the CSS and be done with it, but I thought it would be nice to have some syntax highlighting as well. 

In the Railscast Ryan Bates uses a now-deprecated gem called Abino which is a ruby wrapper for Python's Pygmentize. Now there is [pygments.rb](https://github.com/tmm1/pygments.rb) which is essentially the same thing, but it requires Python and I didn't want to deal with getting Python running on my machine. After some searching I found [Coderay](https://github.com/rubychan/coderay) which is written in Ruby. The documentation on Coderay is...lacking but luckily I found [this](http://allfuzzy.tumblr.com/post/27314404412/markdown-and-code-syntax-highlighting-in-ruby-on-rails) blog post and pretty much copied the code within. So now my helper looks like this:

```ruby
module ApplicationHelper
class CodeRayify < Redcarpet::Render::HTML
  def block_code(code, language)
    CodeRay.scan(code, language).div
  end
end

def markdown(text)
  coderayified = CodeRayify.new(:filter_html => true, 
                                :hard_wrap => true)
  options = {
    :fenced_code_blocks => true,
    :no_intra_emphasis => true,
    :autolink => true,
    :lax_html_blocks => true,
  }
  markdown_to_html = Redcarpet::Markdown.new(coderayified, options)
  markdown_to_html.render(text).html_safe
end
end
```

If you're curious what is going on in that options hash, Redcarpet has very good documentation [here](https://github.com/vmg/redcarpet) which explains what all of the options do and lists some others I didn't use.

One additional caveat with using coderay is that you have to use fenced code blocks and identify the language you are using insead of just indenting.

So there you go, a markdown-friendly blog inside your rails app minus the entire day of googling required to get there. 

PS: if you, like me, always forget markdown syntax might I recommend [Macdown](http://macdown.uranusjr.com/), it shows you what your text will look like once it's converted to HTML. I was using [Mou](http://mouapp.com/), but apparently that is not in development anymore :( and it doesn't support fenced code blocks, Macdown does and for now at least is being actively developed.


---
layout: post
title: "Mermaid Octopress Integration"
date: 2015-05-23 12:00:00 -0700
comments: true
categories: jekyll octopress mermaid
---

[Mermaid](http://knsv.github.io/mermaid/) provides "generation of diagrams and flowcharts from text in a similar manner as markdown".  I wanted to add charting support to the blog (running on [Octopress](http://octopress.org/)), these are the steps I took to get things working.

### download

The best place I found to download mermaid is the [dist folder](https://github.com/knsv/mermaid/tree/master/dist) of the mermaid github repo.


* `mermaid.full.min.js` to `/source/javascripts/mermaid.full.min.js`
* `mermaid.css` to `/source/stylesheets/mermaid.css`


### integrate

I found the (jekyll-mermaid)[https://github.com/jasonbellamy/jekyll-mermaid] plugin and installed it.

In addition to the instructions on the plugin page I had to mannuall add an entry into the Gemfile for rake to pick it up:


``` ruby
group :development do
  gem 'rake', '~> 10.0'
  gem 'jekyll', '~> 2.0'
  # ...snip
  gem 'jekyll-mermaid', '~> 1.0.0'
end
```

### problems encountered

* double dash requires escape (rdiscount related problem)
  * This works `A\-\->B`
  * This will not work `A-->B` 
   
### test diagrams

{% raw %}
<pre>
{% mermaid %}
graph LR;
    A\-\->B;
{% endmermaid %}
</pre>
{% endraw %}

{% mermaid %}
graph LR;
    A\-\->B;
{% endmermaid %}

{% raw %}
<pre>
{% mermaid %}
graph TD;
    A\-\->B;
    A\-\->C;
    B\-\->D;
    C\-\->D;
{% endmermaid %}
</pre>
{% endraw %}

{% mermaid %}
graph TD;
    A\-\->B;
    A\-\->C;
    B\-\->D;
    C\-\->D;
{% endmermaid %}

{% raw %}
<pre>
{% mermaid %}
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John\-->>Alice: Great!
{% endmermaid %}
</pre>
{% endraw %}

{% mermaid %}
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John\-->>Alice: Great!
{% endmermaid %}


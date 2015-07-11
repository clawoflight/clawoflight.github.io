---
layout: post
permalink: /test.html
---

code with liquid:

{% highlight python linenos %}
while is_running():
    task = get_next_task_from_queue()
    if task:
        submit_task_for_processing(task)
    else:
        sleep_for_a_moment()
{% endhighlight %}

code with backticks:

```python
while is_running():
    task = get_next_task_from_queue()
    if task:
        submit_task_for_processing(task)
    else:
        sleep_for_a_moment()
```

<figure>
<img src="/images/hemingway-rewritten_wp_header.jpg">
<figcaption>Single image</figcaption>
</figure>

<figure>
<img src="/images/innsbruck.jpg">
<img src="/images/typewriter.jpg">
<figcaption>2 images?</figcaption>
</figure>

---
layout: post
title: "Drupal 7: Clear Cache Hook"
date: 2014-05-07 12:29:41 +0530
comments: true
categories: [Drupal 7, cache hook]
---

I recently came across a task where I wanted to do some stuff once all Drupal caches are flushed.

I just thought, 'oh, ok. There will be a hook for that.'. But there isn't. On looking at the docs, it seems there is no straight-forward hook for this. They solved this problem (after several requests), in Drupal 8.

All I wanted to do was, once all the caches were cleared, warmup frequently visited pages.

Then I came across this post [How to Ensure that Visitors Always See Cached Pages in Drupal 7](http://tomroelandts.com/articles/how-to-ensure-that-visitors-always-see-cached-pages-in-drupal-7).
While a nice way to handle the issue, I wanted to do this in my module without scripts.

Then I came across this [question at drupal.stackexchange](http://drupal.stackexchange.com/questions/84371/is-there-a-way-to-hook-on-cache-clearing).
I was more interested in the answer which proposed a method where you would write your own database backend upon `DrupalDatabaseCache`.
This answer might not be very clear to some users (such as myself). So I will try to explain it a little in detail here.

What we are doing here is, we are creating a *custom cache implementation* and making drupal use it instead of the default one.
We need the custom cache implementation just to add a hook on cache clear. To create a custom cache implementation, add a class, lets say **DrupalCustomCache**, to your *.module file*. All cache implementations has to *implement* **DrupalCacheInterface**.
```
class DrupalCustomCache implements DrupalCacheInterface {
```
So we take the default implementation from **cache.inc** file from **drupal_root/includes** dir.
Now we copy the contents of the **DrupalDatabaseCache** class to your **DrupalCustomCache**.

To tell Drupal to use our custom cache and not the default one, we have to set a variable. If you want to use you custom cache by default for all cache bins, you should do
```
variable_set_value('cache_default_class', 'DrupalCustomCache');
```
More info on this [here](https://api.drupal.org/api/drupal/includes%21cache.inc/interface/DrupalCacheInterface/7).
Usually this variable can be set in your custom module's *hook_install* and don't forget to delete it in *hook_uninstall*.

The `clear()` method in our **DrupalCustomCache** is where the actual cache tables are cleared. So your logic should be written here. I will not go into much detail about the logic here.

Lets say if you want a hook to be triggered when a particular cache bin is cleared. We can add a simple `if` statement at the last line inside of the `clear()` method and check if the `$this->bin` variable is equal to, say, "*cache_page*". Inside this *if* statement, you can use a simple `module_invoke_all('youmodule_cache_clear');` and then use hook_youmodule_cache_clear.

Thats it for this post. Hope this helps someone. Leave your comments/queries and I will try and answer them.
 
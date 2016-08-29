---
layout: post
asset-type: post
name: Retain &amp; restore recycler view scroll position
title: Retain &amp; restore recycler view scroll position
date: 2015-06-22 17:00:00 +00:00
image:
    src: /assets/img/restore-position.jpg
categories: Android

---

If you come to a situation where you need to retain the scroll position of a recycler view (i.e. configuration change, going back on a certain flow), you might think that you have a retain and restore the scroll manually. Yes, in fact, you have to, but LayoutManager has a convenient API that makes things a bit easier.

{% highlight java %}

LayoutManager.onSaveInstanceState()
LayoutManager.onRestoreInstanceState(Parcelable state);

{% endhighlight %}

These two methods are the ones that you need to use in order to retain and restore the scroll position later on. Now comes the tricky part, In order to work the content of the recycler view has to be loaded before restoring the scroll position.

That means that if you are loading the data asynchronously on your recycler view you will have to keep a reference of the saved stated

First save the current state.

{% highlight java %}

@Override
protected Parcelable onSaveInstanceState() {
   Bundle bundle = new Bundle();
   bundle.putParcelable(SAVED_LAYOUT_MANAGER, recyclerView.getLayoutManager().onSaveInstanceState());
   return bundle;
}

{% endhighlight %}

Then keep a reference from the state previously saved

{% highlight java %}

@Override
protected void onRestoreInstanceState(Parcelable state) {
    if (state instanceof Bundle) {
        layoutManagerSavedState = ((Bundle) state).getParcelable(SAVED_LAYOUT_MANAGER);
    }
    super.onRestoreInstanceState(state);
}

{% endhighlight %}

And restore it after  you've populated the adapter with data.

{% highlight java %}

public void setItems(List objects) {
    adapter.setItems(objects);
    restoreLayoutManagerPosition();
}

private void restoreLayoutManagerPosition() {
    if (layoutManagerSavedState != null) {
        recyclerView.getLayoutManager().onRestoreInstanceState(layoutManagerSavedState);
    }
}

{% endhighlight %}

Voila!

## Personal thoughts

IMO the LayoutManager should keep the restored instance state and apply it once the dataset gets loaded, this would definitely make the code much cleaner and stylish. Instead, what it does under the hood is discard the pendingScrollPosition if the attached recyclerView does not have items when onRestoreInstanceState method is called on the layout manager.

Cross-posted from <a href="http://panavtec.me/retain-restore-recycler-view-scroll-position" target="_blank" >panavtec.com</a>

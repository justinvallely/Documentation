---
title: ASCellNode
layout: docs
permalink: /docs/cell-node.html
prevPage: display-node.html
nextPage: button-node.html
---

<code>ASCellNode</code>, as you may have guessed, is the cell class of ASDK.  Unlike the various cells in UIKit, <code>ASCellNode</code> can be used with <code>ASTableNodes</code>, <code>ASCollectionNodes</code> and <code>ASPagerNodes</code>, making it incredibly flexible.

### 3 Ways to Party

There are three ways in which you can implement the cells you'll use in your ASDK app: subclassing <code>ASCellNode</code>, initializing with an existing <code>ASViewController</code> or using an existing UIView or <code>CALayer</code>.

#### Subclassing

Subclassing an <code>ASCellNode</code> is pretty much the same as <a href = "/docs/subclassing.html">subclassing</a> a regular <code>ASDisplayNode</code>.  

Most likely, you'll write a few of the following:
<ul>
<li>
	<code>-init</code> -- Thread safe initialization
</li>
<li>
	<code>-layoutSpecThatFits:</code> -- Return a layout spec that defines the layout of your cell.
</li>
<li>
	<code>-didLoad</code> -- Called on the main thread.  Good place to add gesture recognizers, etc.
</li>
<li>
	<code>-layout</code> -- Also called on the main thread.  Layout is complete after the call to super which means you can do any extra tweaking you need to do.
</li>
</ul>

#### Initializing with an ASViewController

Say you already have some type of view controller written to display a view in your app.  If you want to take that view controller and drop its view in as a cell in one of the scrolling nodes or a pager node its no problem.

For example, say you already have a view controller written that manages an <code>ASTableNode</code>.  To use that table as a page in an <code>ASPagerNode</code> you can use  <code>-initWithViewControllerBlock</code>.

<div class = "highlight-group">
<span class="language-toggle"><a data-lang="swift" class="swiftButton">Swift</a><a data-lang="objective-c" class = "active objcButton">Objective-C</a></span>
<div class = "code">
  <pre lang="objc" class="objcCode">
- (ASCellNode *)pagerNode:(ASPagerNode *)pagerNode nodeAtIndex:(NSInteger)index
{
    NSArray *animals = self.allAnimals[index];
    
    ASCellNode *node = [[ASCellNode alloc] initWithViewControllerBlock:^UIViewController * _Nonnull{
        return [[AnimalTableNodeController alloc] initWithAnimals:animals];
    } didLoadBlock:nil];
    
    node.preferredFrameSize = pagerNode.bounds.size;
    
    return node;
}
</pre>
<pre lang="swift" class = "swiftCode hidden">
func pagerNode(pagerNode: ASPagerNode!, nodeAtIndex index: Int) -> ASCellNode! {
    let animals = allAnimals[index]
    
    let node = ASCellNode(viewControllerBlock: { () -> UIViewController in
        return AnimalTableNodeController(animals: animals)
    }, didLoadBlock: nil)
    
    node.preferredFrameSize = pagerNode.bounds.size
    
    return node
}
</pre>
</div>
</div>

And this works for any combo of scrolling container node and <code>UIViewController</code> subclass.  You want to embed random view controllers in your collection node? Go for it.

<div class = "note">
	Notice that you need to set the <code>preferredFrameSize</code> of a node created this way.  Normally your nodes will implement <code>-layoutSpecThatFits:</code> but since these don't you'll need give the cell a size.
</div>


#### Initializing with a UIView or CALayer

Alternatively, if you already have a <code>UIView</code> or <code>CALayer</code> subclass that you'd like to drop in as cell you can do that instead.

<div class = "highlight-group">
<span class="language-toggle"><a data-lang="swift" class="swiftButton">Swift</a><a data-lang="objective-c" class = "active objcButton">Objective-C</a></span>
<div class = "code">
  <pre lang="objc" class="objcCode">
- (ASCellNode *)pagerNode:(ASPagerNode *)pagerNode nodeAtIndex:(NSInteger)index
{
    NSArray *animal = self.animals[index];
    
    ASCellNode *node = [[ASCellNode alloc] initWithViewBlock:^UIView * _Nonnull{
        return [[SomeAnimalView alloc] initWithAnimal:animal];
    }];

    node.preferredFrameSize = pagerNode.bounds.size;
    
    return node;
}
</pre>
<pre lang="swift" class = "swiftCode hidden">
func pagerNode(pagerNode: ASPagerNode!, nodeAtIndex index: Int) -> ASCellNode! {
    let animal = animals[index]
    
    let node = ASCellNode { () -> UIView in
        return SomeAnimalView(animal: animal)
    }

    node.preferredFrameSize = pagerNode.bounds.size
    
    return node
}
</pre>
</div>
</div>

As you can see, its roughly the same idea.  That being said, if you're doing this, you may consider converting the existing <code>UIView</code> subclass to be an <code>ASCellNode</code> subclass in order to gain the advantage of asynchronous display.

### Never Show Placeholders

Usually, if a cell hasn't finished its display pass before it has reached the screen it will show placeholders until it has completed drawing its content.

If placeholders are unacceptable, you can set an <code>ASCellNode</code>'s <code>neverShowPlaceholders</code> property to <code>YES</code>.

<div class = "highlight-group">
<span class="language-toggle"><a data-lang="swift" class="swiftButton">Swift</a><a data-lang="objective-c" class = "active objcButton">Objective-C</a></span>
<div class = "code">
  <pre lang="objc" class="objcCode">
node.neverShowPlaceholders = YES;
</pre>
<pre lang="swift" class = "swiftCode hidden">
node.neverShowPlaceholders = true
</pre>
</div>
</div>

With this property set to <code>YES</code>, the main thread will be blocked until display has completed for the cell.  This is more similar to UIKit, and in fact makes AsyncDisplayKit scrolling visually indistinguishable from UIKit's, except being faster.

<div class = "note">
Using this option does not eliminate all of the performance advantages of AsyncDisplayKit. Normally, a cell has been preloading and is almost done when it reaches the screen, so the blocking time is very short.  Even if the <code>rangeTuningParameters</code> are set to 0 this option outperforms UIKit.  While the main thread is waiting, subnode display executes concurrently.
</div>

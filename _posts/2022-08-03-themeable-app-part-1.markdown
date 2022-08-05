---
layout: post
title:  "Writing a Theme-able wxWidgets Application - Part 1"
date:   2022-08-03 22:06:05 -0300
categories: wxWidgets UI/UX C++
---
Our company had a long-standing requirement to support dark mode in a large wxWidgets based application. At the time (and still as I write this), both the Win32 API and wxWidgets had little to no support for dark mode. After much discussion and debate we decided that we were too invested in wxWidgets to switch to another framework. The decision was made to create a library on top of wxWidgets to provide the dark mode elements we needed. Keep reading to learn how we did it. 

![Neutral](/assets/images/aims-themes.png)

### The Problem 
The fundamental problem was that wxWidgets is designed to look like the OS and does not make it easy to change that look. That is to be expected, it is right in their product description. 

> “...wxWidgets gives applications a truly native look and feel because it uses the platform's native API rather than emulating the GUI” - wxWidgets.org

So right from the start we knew we were in for some work.  

### Getting Started 
It did not take long to conclude that if we wanted full control of how the UI looked, we would have to write all our own controls and other UI elements. Of course, we were able to build on much of the work wxWidgets had already done by using base classes like wxControl and wxPanel.  

#### Basic controls 
We started with basic controls like buttons, checkboxes, lists, etc... Although they were lots of work, those items were not overly complex. The basic steps were: 

- Subclass from wxControl. 
- Handle paint events to draw the control. 
- Handle mouse and key board events to add control behaviour and user interaction. 
- Add interface functions for users (other programmers) to interact with the control. 

That list is a bit simplified. You must also deal with state, data storage, responsiveness, performance, and many other implementation details. Still, once the pattern was found, implementing the base controls became straightforward. 

#### Dark Mode 
Now that we had a plan for implementing the basic controls, we needed an approach to support light and dark mode. Our first attempt was to embed colours in the source code using arrays and indexes. This approach worked but obviously required code changes every time we wanted to tweak something. The biggest issue was that our amazing UX/UI designer had to put in a change request every time she wanted to tweak a colour. 

So, we moved from hard coded arrays of colours to colours defined in xml. Our designer produced a palette of named colours and we loaded those colours at runtime. Now our designer could tweak colours and even design whole new themes without having to wait for us developers. Obviously, if we decide to add a new colour to the palette, we need to make code changes, but those occurrences are rare.  

None of this is rocket science, as they say, but not only did we solve our Dark mode requirement, the solution opened us up to multiple theme capability. 

#### Icons 
Like many modern applications, ours makes heavy use of icons and other graphics in the UI. Therefore, our next problem was how to deal with different icons for different themes. For this we discussed several approaches, but in the end came up with a solution like the wxWidgets wxArtProvider class. We initially spent time trying to use wxArtProvider but ultimately we required more functionality than wxArtProvider could provide so we created our own. Like the wxWidgets version, our art provider caches bitmaps once loaded and allows for the addition of classes for loading art, like wxArtprovider::CreateBitmap().  

Currently we store the above-mentioned XML theme file and all images used for icons in a single theme directory. Our art provider, more specifically our art loader, knows where the directory for the current theme is and loads bitmaps from there. Loading from disk is no big deal since we cache bitmaps once loaded. 

Further functionality deals with loading state versions of bitmaps (hover, disabled, etc...) and sizes based on naming conventions. The details of all that are not important to this article. 

#### Switching Theme 
Now that we could create custom theme-able controls we needed a way to load a different theme and have all the windows and controls in the application refresh and use the new theme. Turns out this is the easy part. We'd load the new XML theme file, replace the current colour palette, and then clear the icon cache mentioned above so that new images would get loaded. Next, we just needed to tell all the windows and controls in the application to update themselves. 

wxWidgets makes it easy to create custom events and has a few functions in wxWindow to help enumerate all the windows and controls in an application. We created a simple "theme changed" event and sent it to every window in the application. Each window must simply respond to the "theme changed" event and update itself. Since colours are indexed and icons are loaded when requested, in most cases all a control or window needs to do is redraw itself. This last part was made even easier by having base classes for controls, panels, and windows that already handle this event. 

### Conclusion (Part 1)
Now that we have all the basics in place it's time the tackle the bigger issues, complicated controls (like multi-column lists), menus, scrollbars, etc... I'll cover those items in Part 2 in a couple weeks.

---
layout: post
title:  "Writing a theme-able wxWidgets Application - Part 2"
date:   2022-08-26 16:06:05 -0300
categories: wxWidgets UI/UX C++
---

### The Problem Children
In [Part 1]({% post_url 2022-08-03-themeable-app-part-1 %}) I discussed how we got the basics in place for our theme-able wxWidgets application. I covered basic controls, setting up themes, dealing with icons and notifing the app about theme changes. Well, that was it for the easy part, next we had to tackle the more difficult controls and UI elements. Elements such as scrollbars, multi-column lists (with headers), menu and title bars proved to be the most challenging things to deal with.

#### Scrollbars
Writing a scrollbar control itself was not particularly difficult. However, getting panels in wxWidgets to use it instead of the system scrollbar was the tricky part. After trying a few things and spending many hours searching the internet (where the general advise was to not bother), the only solution we could come up with was to put the panel we wanted to scroll in another panel with a wxFlexGridSizer. The scrolled panel goes in the one and only growable cell in the sizer and the horizontal and vertical scrollbars go in fixed size cells. Then, in the scrolled panel, we had to make sure scroll styles were not enabled, override the SetScrollbar() function to make sure the system scrollbars would not appear, and our scrollbars were updated properly. Luckily for us, wxWidgets provides a wxScrolledHelper class that does much of the work.  

We ended up writing a templated class much like wxScrolled<T> to help with this. Our class created the owner for panel <T> internally and took care of all the function overloading. In the end this worked well, but it still felt very much like a hack, and we will always be open to fresh solutions. 

#### Multi-Column Lists 
We had made heavy use of both wxListCtrl and wxDataViewCtrl (wxDataViewListCtrl mostly) and therefore needed a solution for a theme-able multi column list control. Our first attempt was to write our own. While this worked ok, we quickly realized that reimplementing all the required functionality in the controls mentioned above would take an awfully long time, not to mention the number of bugs we would have introduced. We therefore needed a way make the existing list controls theme-able. 

First, since wxListCtrl is an implementation of the Win32 List Control and modifying all parts of it is impossible, we decided not to use it at all. We swapped out all occurrences of wxListCtrl for wxDataViewListCtrl (since we used wxListCtrl only in details mode, this was possible). 

That left wxDataViewListCtrl. There were three issues with that control we had to find solutions for: its scrollbars, its contents, and its header. The scrollbars were handled mostly the same as above so no need to go into that one.  

On Windows, the wxDataViewListCtrl is a generic control so we were able to create our own wxRenderer class to draw the items (cells). This handled the contents of the control and proved useful for a few other things. 

The last thing to deal with was the header of wxDataViewListCtrl. Even though the wxDataViewListCtrl is the generic control on Windows, wxWidgets still used the Win32 header control internally. Had they used the generic wxHeader control internally we could have handled the theming of the header via our wxRenderer class mentioned above. As it turned out, our only option was to make wxWidgets use the generic wxHeader control on Windows. This was our only modification to wxWidgets itself. 

#### Menu bar and Menus
We thought this was going to be a tricky one, but it turned out to be not so bad. We implemented menus using wxPopupTransientWindow. There were a few tricky bits to work around when it came to handling submenus (multiple windows), but wxPopupTransientWindow handled much of the work. We handled all of the drawing and user interaction the same as we did for any other control. 

The menu bar proved to be just as straightforward. We implemented it in a wxPanel (or our base subclass of it) and again, handled the drawing and interaction ourselves. The placement of the menu bar was the next thing to figure out. Since we use wxAUI this too was not particularly difficult. We simply placed our menu bar panel in an wxAUI toolbar style pane docked at the top of the application main window. 

#### Title bars
Theming the title bars and borders of windows and dialogs could only be done by handling the Win32 non-client area event messages directly. Handling those messages is beyond the scope of this post. Watch for a future post on this topic.

#### wxAUI
We use wxAUI so we also needed to override most of the drawing it does. wxAUI makes that easy by allowing you to provide you own art providers. For this we had to override wxAuiDockArt and wxAuiGenericTabArt to provide drawing using our own theme details.  

### Conclusion 
wxWidgets may not have been the easiest library to work with when adding theme support, but it was certainly doable. If I had to do it all again knowing what I know now, I would consider doing it differently. If I had to do it in wxWidgets again I would look more closely at the wxUniversal port. I know it is not complete and may even be abandoned (that is not clear to me) but given the effort that went into creating all our own controls, it may have been a good idea to extend or revive that effort. While I am not sure it would be a good idea, I would spend more time exploring it. If I had more freedom of choice with the library, I would choose one that already had theme support or at least had more support for custom control rendering. 

In the end we managed to create a completely theme-able application in wxWidgets. It was a ton of work and took a little more than two years to complete. I, as the main developer on the task, was lucky to work for an organization committed to this goal and who provided the support needed to get it done. I did not spend 100% of my time on task (hence the 2+ years), but it was my focus. We were also fortunate to have a talented UI/UX designer providing the design and feedback along the way. 
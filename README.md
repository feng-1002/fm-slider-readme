## Overview
This is a lightweight, vanilla JS, CSS-optimized, simple slider that relies on CSS scrollsnapping and a smoothscroll polyfill to provide native browser scrolling in nearly all slideshow modes.

Goals and traits of this slider include:
1. Minimal DOM manipulation
2. Native browser scrolling
3. Partial existing markup for the sake of low content layout shift and overall performance
4. Easy-to-add breakpoint-specific settings
5. Smooth integration with Shopify section settings
6. Extremely modular setup for better minification and flexibility
7. Fully accessible while also being flexible for multiple possible layouts/design

## Example Code
While some basic examples are listed below, there are a lot of possible features and functionality to this slider! For that reason, the section called `example-sliders.liquid` has been created in the starter theme codebase. It includes both methods of initialization, has all possible settings included for updating in both the corresponding JS file and the section settings, and is meant to be a complete working example of the slider.

This should be referred to when a detailed example is needed! This liquid section should also be kept up to date as slider updates are made, so that it reflects all current settings, markup, and functionality.

Example files include the following:
* [`_src/scripts/sections/example-sliders.js`](https://github.com/fuelmade/2.0-starter-theme/blob/main/_src/scripts/sections/example-sliders.js)
* [`_src/styles/sections/example-sliders.scss`](https://github.com/fuelmade/2.0-starter-theme/blob/main/_src/styles/sections/example-sliders.scss)
* [`sections/example-sliders.liquid`](https://github.com/fuelmade/2.0-starter-theme/blob/main/sections/example-sliders.liquid)

## File Structure

Note: Because this slider relies so much on CSS to provide functionality, the script will first load the core slider styles and only initialize sliders when that stylesheet is loaded. That stylesheet will only load once per page. All of this happens automatically through the script, and all that's required is that the stylesheet exists in `assets`, and that the link to that asset exists as the value for `window.fmSliderStylesheetLink`. Note that that data is typically set in `snippets/theme-window-data.liquid`.

* **fm-slider/scripts**
	* **defaults:** Default slider settings, and default slider class names that are added through the script.
  * **events:** All event listeners and the event bus that is used for slider-specific events.
  * **features:** All features that can be optionally removed or added. (Such as autoplay, infinite, etc.) These components are added into the slideshow instance through `fm-slider/components/index.js`. Removing any component from that file will result in the code being excluded during final minification.
  * **run:** Functionality used when the slider starts running and is finished building, including the updating of dots, the go to slide functionality, the the Intersection Observer that controls which slides are active and updates the current index/dots on native scrolling.
  * **setup:** All core setup functionality, including slider building, registration, and settings.
  * **utilities:** Small helper functions used throughout the entire slider. Some of these exist to ensure that the final minified file is as small as possible, others are there to make DOM updates in a variety of contexts.
  * **index.js** Contains the final exported slider class.

* **fm-slider/styles**
	* **components:** Includes all slider component styles.
	* **extends:** Any and all invisible, extendable classes used by the slider stylesheets.
	* **index.scss** The final stylesheet with all forwarded components.

## Initialization
Sliders can be initialized in one of two ways, either through a data-attribute on the slideshow itself, or through an external script.

### Data Attribute
Any element with the data attribute `data-fm-slider`, when it has a JSON object as its value and is either surrounded by single quotes or is escaped using the `escape` liquid filter, will be initialized automatically (in the case of using within Shopify sites, the file that initializes is in `_src/scripts/fm-slider-initialize.js`, as it relies on theme lazyloading).

The below example assumes that the slideshow is being built using Liquid. 

```html
{% capture slideshowOptions %} 
  {
    "dotsShow": true,
    "slidesToShow": 2,
    "carousel": true,
    "responsive": [
      {
        "breakpoint": "48rem",
        "settings": {
          "slidesToShow": 1,
          "carousel": false
          }
      }
    ]  
  }
{% endcapture %}

<div data-fm-slider="{{ slideshowOptions | escape }}">

  <div class="js-fm-slider-slides">
    {%- for num in (1..3) -%}

      <div class="js-fm-slider-slide">
        Slide Content
      </div>

    {%- endfor -%}
  </div>

  <fieldset>
    <legend class="sr-only">
      Slider Controls
    </legend>

    <button type="button" class="js-fm-slider-prev">
      Previous Slide
    </button>

    <div class="js-fm-slider-dots"></div

    <button type="button" class="js-fm-slider-next">
      Next Slide
    </button>

  </fieldset>
</div>
```

### New Javascript Object
New sliders can be initialized within any other JS file. Additionally, slideshows will only be initialized once, no matter how many times or at what time initialization is attempted. This prevents duplication and conflicts. This means that new slideshow instances can be created wherever needed, because if the slideshow has already been initialized, the current slideshow class will be returned and the publicly available functions will be usable.

```javascript
import { FmSlider } from "../../fm-slider/scripts"

const slideshow = document.getElementById('mySlideshow')
const settings = {
	dotsShow: true,
	slidesToShow: 1
}

new FmSlider(slideshow, settings)
```

## Public Class Methods

| Method                                | Paramater                             | Description
| ------------------------------------- | ------------------------------------- | -------------------------------------
| setupSlideshow                        | none                                  | Sets up and runs the slideshow from scratch. Useful if slider content will be changed after initialization. An example would be a variant-specific product gallery whose images change on variant selection. This method could be called on variant change to refresh the gallery.
| goto                                  | index {Integer} - The new slide index | Triggers the slider to go to the passed index
| advanceSlideshow                      | count {Integer} - How many slides to advance | Triggers the slider to go to the next slide. Is used by the autoplay feature when enabled.
| initialize                            | none                                  | Initializes the slideshow in keeping with current settings.
| destroy                               | none                                  | Destroys the slideshow instance, regardless of current settings.

### Example Usage

Pulling from the example mentioned above, for the product page variant-specific gallery, here's an example of how these methods can be used within a JS file. In this example, the slider is destroyed on desktop, and initialized on mobile. The functions and methods run below are being run on variant change. (This example is pulled almost directly from FM theme's [PDP on Natural Dog](https://shop.naturaldogcompany.com/products/skin-soother-tin?preview_theme_id=127336022109), which at the time of this writing is not live yet.)

```javascript
const sliderSelector = ".js-product-gallery"

const settings = {
    destroy: true,
    selectors: {
        slide: '.js-product-gallery-slide-visible'
    },
    responsive: [
        {
            breakpoint:"48rem",
            settings: {
                dots: true,
                dotThumbnails: true,
                ariaLabel: "Product",
                slidesToShow: 1,
                carousel: false,
                destroy: false,
            },
        },
    ],
}

const sliderInstance = new FmSlider(sliderSelector, settings)

// Find all images associated with the current variant,
// and only include the js-product-gallery-slide-visible class on those images
filterImages(slideshow, variant.id)

// Reset slideshow with filtered images, if it isn't destroyed
if (!sliderInstance.settings.destroy) sliderInstance.setupSlideshow()
```

### Autoplay-Specific Public Methods

If autoplay is included as a component of the slider, and is enabled at any breakpoint, the methods below are available. The autoplay class is accessed through `FmSlider.components.autoplay`.

| Method                                | Paramater                             | Description
| ------------------------------------- | ------------------------------------- | -------------------------------------
| start                                 | none                                  | Starts autoplay
| stop                                  | none                                  | Stops autoplay
| initialize                            | FmSlider {Object} - The current slideshow instance | Initializes autoplay based on current settings, and adds or removes event listeners to handle when autoplay stops and re-starts.

## Removable Components (Slider Features)
Since this slider is meant to be as modular as possible, much of its functionality can be added or removed in a very modular way. All "add-on" components live inside `fm-slider/scripts/features/components`, and are loaded in `fm-slider/scripts/features/index.js`. Any feature can be removed from the `activeComponents` object. If a component is removed, then its code will not be imported and will not be a part of the final compiled script. It's recommended that any unnecessary components be removed to decrease the overall slider script weight.

Below is a list of all current existing components. Note that even if these components are included, some of them still have to be turned on or off in each slider's settings in order to work.

| Component        | Has to be enabled in slider settings | Description | 
| ---------------- | ------------------------------------ | -------------------------------------------------------------------- |
| Autoplay         | true | Provides autoplay, including the functionality of any play/pause button that's included for the slider. This is a class within the larger fmSlider class that can be accessed outside of itself.   |
| breakpoints      | false | Will nearly always be used by any slider. If a slider has responsive settings, it will listen for a change in breakpoint and re-initialize the slider based on those breakpoint-specific settings. |
| controls         | false (If the element itself exists in the markup and can be found via its selector, functionality will automatically be enabled) | Provides functionality for dots and navigation buttons. |
| infinite         | true | Provides infinite functionality. (Note that infinite functionality will never be enabled on touch devices, so that native browser scrolling can be used.) |
| variant          | false | Provides an event listener that listens for the custom event `variantUpdate`. As long as an object is included with the custom event data, that has an `index` property, it will then advance the slideshow to that specified index. |
| fade             | true | Provides fading functionality as opposed to standard scrolling slide functionality. Fade will never be enabled on touch devices, so that native browser scrolling can be used. Note that fade cannot be used alongside the `carousel` setting, as it will only fade in the current active slide, and the carousel setting, when enabled, shows parts of inactive slides (meaning that those would not be visible at all when fade is enabled). |

## Settings
Sliders can all have their own settings. Breakpoint-specific settings can also be included, and the script uses the window.matchMedia method to test for window size using max-width. This is more performant than using a resize event listener.

The available settings and their default values are as follows:

| Option          | Type              | Default Value  | Description                                                                                                                                                                                                                                                       |
| --------------- | ----------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ariaLabel\*     | String or Boolean | false          | The aria label to assign to the slider. Label will only be applied if the slideshow is not currently destroyed. If no string is provided, the name will be set to "Slideshow (number)" where number is the unique slideshow ID, which starts at 1 and increments. |
| autoplay        | Boolean           | false          | Whether to enable autoplay or not. Autoplay will not run regardless of this setting IF the user prefers reduced motion.                                                                                                                                           |
| autoplaySeconds | Integer           | 3              | How many seconds to wait until automatically going to the next slide                                                                                                                                                                                              |
| carousel        | Boolean           | false          | Whether to enable carousel mode. If true, will show slides peeking in from the side(s)                                                                                                                                                                            |
| carouselOffset  | Integer           | 10             | If carousel mode is enabled, this sets the width of the offset (the amount of the next/previous slides that will be visible). The value is used as a percentage within the script.                                                                                |
| destroy         | Boolean           | false          | Whether to destroy the slider instance                                                                                                                                                                                                                            |
| dotsShow        | Boolean           | true           | Whether to show dots/thumbnails                                                                                                                                                                                                                                   |
| dotsVertical    | Boolean           | false           | Whether the dots are scrolling vertically                                                                                                                                                                                                                       |
| dotThumbnails | Boolean           | false          | Whether to show dots as thumbnails. Requires an image inside the slide. Auto-creates these thumbnail dots.                                                                                                                                                                                     |
| dotThumbWidth   | Integer           | 200            | The width of the dot thumbnail image, if thumbnails are enabled                                                                                                                                                                                                                    |
| dotThumbHeight  | Integer           | 200            | The height of the dot thumbnail image, if thumbnails are enabled                                                                                                                                                                                                                    |
| fade            | Boolean           | false          | Whether to fade slides instead of scrolling. Fade will not be enabled regardless of this setting if the user is on a touch device, for the sake of using native scrolling.                                                                                        |
| firstSlideIndex | Integer           | 0              | Which zero-based index to use for the first slide. |
| infinite        | Boolean           | false          | Whether to enable infinite mode. Infinite will not be enabled regardless of this setting if the user is on a touch device, for the sake of using native scrolling.                                                                                                |
| offsetLeft      | Integer           | 0              | How wide of an offset to add to the left of the slider. This setting has no effect when infinite is enabled. Set to zero to show no left offset.                                                                                                                  |
| offsetRight     | Integer           | 0              | How wide of an offset to add to the right of the slider. This setting has no effect when infinite is enabled. Set to zero to show no right offset.                                                                                                                |
| responsive      | Array             | not applicable | An array of objects containing breakpoints and corresponding settings. Details below.                                                                                                                                                                              |
| selectors\*     | Object            | not applicable | Selectors to use for slideshow content. Details below.                                                                                                                                                                                                            |
| slidesToShow    | Integer           | 1              | How many slides to show at one time                                                                                                                                                                                                                               |
| slidesToScroll  | Integer           | 1              | How many slides to scroll by. This only affects the slideshow functionality when infinite is enabled, or when controlling the slideshow through navigation buttons. Otherwise, native browser scrolling controls movement.                                        |


*These settings cannot be set a responsive, breakpoint-specific level. They can only be set in the base settings.

### Setting Selectors
The following settings are available for setting selectors. All values are strings. These should only be set at the root settings level, not at a responsive/breakpoint-specific level.

| Option           | Default Value           |
| ---------------- | ----------------------- |
| dots             | ".js-fm-slider-dots"    |
| dot              | ".js-fm-slider-dot"     |
| slides           | ".js-fm-slider-slides"  |
| slide            | ".js-fm-slider-slide"   |
| prevBtn          | ".js-fm-slider-prev"    |
| nextBtn          | ".js-fm-slider-next"    |
| autoplayBtn      | ".js-fm-slider-autoplay-btn" |
| autoplayBtnText  | ".js-fm-slider-autoplay-btn-text" |

### Responsive Settings

The following settings are available to be used within an object for the responsive array in settings.

| Option           | Type     | Description                                                                                      |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------ |
| breakpoint       | String   | Any valid CSS media query value that can be paired with `max-width:`, used by window.matchMedia to determine the current breakpoint. Uses `max-width` to test for a matching breakpoint. |
| settings         | Object   | All settings for this breakpoint (can use any of the settings listed above, unless otherwise noted). For any settings not added here, the defaults will be used. |

### Example Settings

The following is an example of the settings JSON string. IMPORTANT: Because the slider uses a max-width media query, and increments through the responsive array, smallest breakpoint values should be listed first so that the slider script finds the correct breakpoint settings, as in the example below - the 48rem breakpoint is listed before the 90rem breakpoint.

```json
{
	"dotsShow": true,
	"slidesToShow": 2,
	"carousel": true,
	"responsive": [
		{
			"breakpoint": "48rem",
			"settings": {
				"slidesToShow": 1,
			}
		},
		{
			"breakpoint": "90rem",
			"settings": {
				"infinite": true,
				"slidesToShow": 4
			}
		}
	],
	"selectors": {
		"prevBtn": ".my-prev-btn-selector",
		"nextBtn": ".my-next-btn-selector"
	}
}
```
## Event Emitting
A feature of the slider is the Events class on the slider object, the code for which is located in `fm-slider/scripts/events/event-bus.js`. These custom event emitters allow the slider to mark specific times within the slider's lifespan, so that other scripts can run at those times. For example, the infinite component plugs into the goto.before event to execute the infinite effect before the main goto function fires and the slides move. This allows slider components (such as the infinite feature) to be added or removed as needed, without having to edit the core code.

### Event Emitter Public Methods

| Method                                | Paramaters                            | Description
| ------------------------------------- | ------------------------------------- | -------------------------------------
| on                                    | customEvent {String} - The event name <br>callback {Function} - The callback to run when the timing is fired.                                | Adds the callback to the custom event, whether the event existed previously or not.
| emit                                  | customEvent {String} - The event name <br>sliderInstance {Object} - The current slider instance <br>data {*} - Optional data to be passed to the callback. Used by the goto function to specify the new index. | Emits the event and runs all associated callbacks.

### Core Events
While any new event can be added and emitted wherever neccessary, the core ones are listed below.

| Event Name                            | Description                 
| ------------------------------------- | ----------------------------------------------------------------------------------------
| build.before                          | Is emitted before the slideshow builds content (including offsets and dots). Is only run when the slideshow if first initialized after page load.
| build.after                           | Is emitted after the slideshow build process is complete.
| index.changed                         | Is emitted once a new slide has been shown and the index has been updated accordingly. Emitted regardless of whether a control was used, or native scrolling was used to show a new slide.
| initialize                            | Emitted each time the slider is initialized, which occurs after initial build, and when each breakpoint is changed (if the slider settings include breakpoint-specific settings)
| goto.before                           | Emitted before the goto function enables the slider to go to the new slide. Used by the infinite component to handle the infinite effect.
| goto                                  | Emitted when the slider needs to go to a new slide. This only happens when triggered by a control (when nav buttons are clicked, dots are clicked, etc.), not when native scrolling is happening.
| goto.after                            | Emitted after the slide has finished scrolling and the slider has stopped moving.
| slides.activate                       | Emitted each time the slider is initialized, and each time the slider is natively scrolled. 
| update.dots                           | Emitted each time the slider is initialized, and each time the slider is natively scrolled. 

### Plugging Into Events Externally

If you need to inject your own functionality at certain slider timings, the `Events` class that exists on the slider object can be directly used! For example:

```javascript
import { FmSlider } from "../../fm-slider/scripts"

const slideshow = document.getElementById('mySlideshow')
const settings = {
	dotsShow: true,
	slidesToShow: 1
}

const slider = new FmSlider(slideshow, settings)

slider.Events.on("goto", sliderInstance => {
  console.log(sliderInstance)
})
```

Note that if you've already initialized the slideshow but don't have it stored yet as a variable that can be used, you can retrieve the current instance like this:

```javascript
const slider = new FmSlider('#mySlideshow')
```

What this does is if the slider has not already been initialized, it will be at this point - otherwise the slider instance will be returned and will be available for modification!

## Accessibility Features

This is a summary of the features of the slider and all the ways in which it seeks to not only meet accessibility guidelines, but provide an intuitive and smooth experience for all users.

While the W3C has some guidelines on carousels, some of those guidelines conflict with each other, and there are just a few of them that may be problematic from a real-world standpoint. The accessibility features here attempt to bridge the gap between the W3C guidelines and data from experienced accessibility experts.

### Markup Structure

* **Slides:** While it might seem correct to put all slides inside an HTML list, this would result in screen readers announcing an incorrect number of total slides, since inactive slides are hidden from screen readers and keyboards. Instead, slides are contained in a regular div, and themselves use div tags, while their numbered position in the total slides is added to an aria-label on the slide's div container. Reference: [Why lists should be avoided within sliders](https://dev.to/jasonwebb/how-to-build-a-more-accessible-carousel-or-slider-35lp#:~:text=Avoid%20using%20list,this%20use%20case)

* **Controls:** All the controls (except for the autoplay button) should generally be inside a fieldset, with a legend that signifies that this is a group of carousel controls.

### Navigation Buttons

* **Button Functionality:** Next and previous buttons are included, so users can control the slider. True button elements are used here, so that all devices can discover them. Text is included inside the button (whether visibly hidden if using an icon, or visible to all) so that the button can be read programmatically.
* **Button Placement:** Buttons are inside a fieldset, together with dots/thumbnails (if dots are enabled). They can be positioned anywhere using CSS, as long as their tabbing order matches their visual placement.

### Dots/Thumbnails

- **Dot Functionality:** Whether dots are used (and generated by the script), or thumbnails are being used (and already exist in the markup), they will always be radio buttons. General reference to this pattern here: [Accessibility Developer Guide: Carousel Widget](https://www.accessibility-developer-guide.com/examples/widgets/carousel/) This is done for a few reasons:
    - **Less Tabbing:** Instead of forcing the user to tab through all dots (which would be the case if the dots were buttons), hitting tab once will move them past the dots.
    - **Radio Pattern Matches Dots Pattern:** The native HTML radio pattern matches carousel dots exactly. i.e. only one radio button can be selected at once, just as only one slide is the current slide; radio buttons are built to work very well even if there’s a large number of them, just as there could be a large number of dots in a slider. Additionally, users and their devices have the benefit of being immediately familiar with a radio button and what it does/how it use it.
- **Maintaining Focus:** While the W3C guidelines [specify that after choosing a dot, focus should be transferred to the corresponding slide](https://www.w3.org/WAI/tutorials/carousels/functionality/#focus-the-selected-carousel-item), this likely goes against [WCAG 3.2.2](https://www.w3.org/WAI/WCAG21/Understanding/on-input.html) because it’s an unexpected change of focus and context. Not only that, but transferring focus to the active slide could result in an unexpected page shift, if the slide is partially out of view - creating a jarring and unexpected effect. Additionally, the FM slider is set up so that the only slides that can be interacted with are the currently visible slides, so there’s no risk of the user tabbing to slides and getting lost inside them. So in this case, when a user chooses a dot, focus will remain on that dot until they decide to tab away.

### Arrow Keyboard Controls

- **Native Scrolling:** Unless a slideshow is in infinite mode or fade mode, native scrolling is enabled for the slides container. The container will have a tabindex of 0 to allow keyboards to reach it easily and initiate scrolling.
- **Custom Controls:** Beyond native scrolling, there are no custom controls for keyboard arrow keys. This is so as not to disrupt the native functionality of arrow keys (such as for scrolling), especially considering the fact that screen reader users use arrow keys to navigate with a virtual cursor. Reference: [why custom arrow controls shouldn’t be used](https://dev.to/jasonwebb/how-to-build-a-more-accessible-carousel-or-slider-35lp#:~:text=Don%E2%80%99t%20use%20custom%20keyboard%20controls%2C%20like%20arrow%20keys%20for%20navigation.%20These%20just%20confuse%20real%20keyboard%20users%2C%20and%20are%20probably%20going%20to%20be%20missed%20entirely%20by%20screen%20reader%20users%20since%20they%20already%20use%20their%20arrow%20keys%20for%20navigating%20with%20their%20virtual%20cursor).

### Autoplay Control

- **Pausing and Playing:** The autoplay button enables the user to start or stop the autoplay feature. However, it’s important to note that what this does (when the pause button is clicked) is it signals autoplay to never start at all. Autoplay is not only paused when the button is clicked - it is also paused whenever the slider receives focus, a mouseover, or touchstart (because it is assuming that the user is interacting with it - therefore there should be no automatic movement). Pressing play on the button will allow autoplay to resume only when the user is not currently focused on, mousing over, or touching the slider.
- **Control Placement:** [The button should come before any other slider content](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/#:~:text=If%20present%2C%20the,be%20easily%20located.), so the user can immediately pause it if they need to.
- **Reduced Motion:** By default, if the user prefers reduced motion, autoplay will never start, regardless of the slideshow’s current settings.
- **Control Text:** In addition to a selector for the autoplay button, there is a separate selector for the button text. This is updated when the user clicks on the button, and switches from saying “Stop autoplay” to “Start autoplay.” Screen readers will announce this text when the button receives focus as well as when the text is changed, so that users know this button toggles autoplay.

### Aria Features

- **Overall Slider Container:** The main slider container has its role set to region, it includes an aria-roledescription of carousel, and it includes an aria-label that is either set through settings or through the script itself if no label is provided. Because it’s explicitly set to have a role of region, it should only ever be a div tag, and not use any other HTML tags with implicit roles.
- **Individual Slides:** Each slide has a role of group, with an aria-roledescription of slide, and a label that signifies its placement in the overall slide group. This label can either be custom, added through the `data-slide-label` attribute, with placement data added (i.e. My unique label, 1 of 10), or if no custom label is provided, it will be only the placement data (i.e. 1 of 10). Note that [the word “slide” doesn’t need to be included](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/#:~:text=Note%20that%20since%20the%20aria%2Droledescription%20is%20set%20to%20%22slide%22%2C%20the%20label%20does%20not%20contain%20the%20word%20%22slide.%22), because it is already the role description. If the slide is inactive, it will have the aria-hidden attribute, and all of its interactive content (if there is any) will be set to have a tabindex of -1.
- **Next/Prev Buttons:** These have aria-controls attributes, with the value being the ID of the slides container.
- **New Slides Announced:** As long as autoplay is not enabled, the slides container has the aria-live attribute, with its value set to polite. It will announce new slides (and only the new slides) as they enter the viewport, through the aria-atomic attribute being set to false. Reference: [W3C announce new slides](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/#:~:text=Optionally%2C%20an%20element,NOT%20automatically%20rotating).

### General Functionality

- **Active/Inactive Slides:** Any slides that are less than 90% visible within the slider are considered inactive, and will be inaccessible to screen readers and keyboards (through the aria-hidden attribute and the setting of interactive elements to a tabindex of -1).

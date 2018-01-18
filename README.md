# mySociety analytics helper library

Some helpful bits of JavaScript for tracking events in Google Analytics.

Nicely handles the case where Google Analytics has been blocked by the client,
or hasn’t loaded properly.

Requires “Universal Analytics” (`…/analytics.js`), `ga()`) rather than any
previous version of Google Analytics such as “Asynchronous Analytics”
(`…/ga.js`, `_gaq.push()`).

## `analytics.trackEvent(params)`

*Arguments:*

* `params` should be an object of settings for the `ga('send')` command,
   such as `hitType`, `eventAction`, or `eventCategory`. Consult the
   [Google Analytics Field Reference](https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference)
   for a full list of parameters to the `ga('send')` command.

*Returns:* A jQuery Deferred promise. The promise is resolved when the `ga()`
call completes (`hitCallback`) or when 2 seconds have elapsed, whichever
happens first.

*Example:*

    $(document).on('submit', '.homepage-try', function(e) {
        analytics.trackEvent({
            eventCategory: 'homepage-try__search',
            eventAction: e.type
        });
    });
    
    $('.close').on('click', function(e){
        analytics.trackEvent({
            eventCategory: 'close-modal'
        });
    });

## `analytics.trackEvents(listOfEvents)`

Useful for issuing a number of different `ga('send')` commands as a result of
a single interaction with an element on the page.

*Arguments:*

* `listOfEvents` should be a list of objects, where each object is something
   that would be suitable for passing to `analytics.trackEvent` (ie: an object
   of parameters for a `ga('send')` command.

*Returns:* A jQuery Deferred promise. The promise is resolved when all of the
child `trackEvent` calls are resolved.

*Example:*

    $(document).on('submit', '.search-form', function(e) {
        analytics.trackEvents([
            {
                eventCategory: 'search-submitted',
                eventLabel: $('#q').val()
            }, {
                eventCategory: 'header-interaction',
                eventAction: e.type,
                eventLabel: '.search-form'
            }
        ]);
    });

## `analytics.trackLinkClick(e, params)`

Useful for tracking clicks on a link that would normally lead to a new page,
and waiting for the event to be tracked before continuting to the new page.

*Arguments:*

* `e` should be a jQuery Event object, most likely provided by the `$.click`
   handler from which you are calling `.trackLinkClick`. We use the Event
   object to detect whether the user held down a modifier key while clicking
   (which means the link will open in a new browser tab, and we no longer need
   to prevent the default browser navigation behaviour, or modify the current
   window’s location when we’re done).
* `params` should be an object suitable for passing to `analytics.trackEvent`.

*Returns:* A jQuery Deferred promise, from `analytics.trackEvent`, although
you are unlikely to need to use this promise, since `.trackLinkClick` adds
its own URL redirection callback to the resolution state of the promise.

*Example:*

    $('a.pricing').on('click', function(e){
        analytics.trackLinkClick(e, {
            eventCategory: 'navigate-to-pricing-page'
        });
    });

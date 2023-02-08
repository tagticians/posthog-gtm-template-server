# PostHog Custom Events template for Google Tag Manager [Server]

This template will help you setup custom events for PostHog using **Google Tag Manager Server Side**, allowing you to map event and user parameters for their respective events.

If you are looking for the Google Tag Manager Template for the Web [check out this repository](https://github.com/tagticians/posthog-gtm-template).

> **⚠️** This template **will only work when PostHog’s client-side snippet is implemented** either hardcoded or through a Custom HTML tag. This is a limitation of the PostHog Javascript SDK.

> ✅ The benefit of hardcoded or Custom HTML tag approach is that it **will allow you to:**
> - **[Recommended]** Disable client-side pageview events.
> - **[Optional]** Disable autotracking capabilities.
>  - Set and maintain important client-side variables in the PostHog cookie, including:
>     - Device ID (device_id)
>     - User ID ($distinct_id)
>     - Session ID ($sesid)
>     - referrer ($referrer)
>     - referring_domain ($referring_domain)
>     - Search Engine ($search_engine)
>     - Active Feature Flags ($active_feature_flags)
>     - Enabled Feature Flags ($enabled_feature_flags)
>     - Feature Flag Payloads ($feature_flag_payloads)
>     - Session Recording Enabled Server Side ($session_recording_enabled_server_side)
>     - Any $set or $set_once variables
Learn more about [available configuration options](https://posthog.com/docs/integrate/client/js#config).

## Requirements

1. Google Tag Manager client-side and server-side implemented.
2. GA4 Configuration tag in the client-side container has 'Send to server container' enabled.
3. PostHog client-side base script is implemented.

## GTM Client-side preparations:

1. In GTM, create a Constant variable with Project API Key which can be found on your PostHog’s Project Settings page.
2. Grab PostHog's Javascript SDK snippet from the same Project Settings page.
3. In GTM, create a Custom HTML tag and paste the snippet.
4. Replace the Project API Key string with the Constant variable you just created. This will save you time in the future and prevent unwanted errors when you need to replace your API key.
5. Add `capture_pageview:false` [configuration to PostHog's script](https://posthog.com/docs/integrate/client/js#config) to prevent the pageview from being sent client-side. Your Custom HTML code should look something like this:

```html
<script>
! function(t, e) {
    var o, n, p, r;
    e.__SV || (window.posthog = e, e._i = [], e.init = function(i, s, a) {
        function g(t, e) {
            var o = e.split(".");
            2 == o.length && (t = t[o[0]], e = o[1]), t[e] = function() {
                t.push([e].concat(Array.prototype.slice.call(arguments, 0)))
            }
        }(p = t.createElement("script")).type = "text/javascript", p.async = !0, p.src = s.api_host + "/static/array.js", (r = t.getElementsByTagName("script")[0]).parentNode.insertBefore(p, r);
        var u = e;
        for (void 0 !== a ? u = e[a] = [] : a = "posthog", u.people = u.people || [], u.toString = function(t) {
                var e = "posthog";
                return "posthog" !== a && (e += "." + a), t || (e += " (stub)"), e
            }, u.people.toString = function() {
                return u.toString(1) + ".people (stub)"
            }, o = "capture identify alias people.set people.set_once set_config register register_once unregister opt_out_capturing has_opted_out_capturing opt_in_capturing reset isFeatureEnabled onFeatureFlags".split(" "), n = 0; n < o.length; n++) g(u, o[n]);
        e._i.push([i, s, a])
    }, e.__SV = 1)
}(document, window.posthog || []);
posthog.init({{CONST - Posthog Project API Key}}, {
    api_host: 'https://eu.posthog.com',
    capture_pageview: false,
    capture_pageleave: false
})
</script>
```

   > ⚠️ Remember to use the correct `api_host` address linked to your account.

   > ⚠️ If you also want to **disable the autocapture feature** make sure to add `autocapture:false` to the post.init options object.

6. Add an All Pages trigger (or any other applicable trigger) to the Custom HTML tag, then Save.
7. In GTM, create a Cookie variable to grab PostHog cookies. The cookie that is set by PostHog, and that contains valuable information, has a name that looks like this: `ph_<api_key>_posthog`.
8. In the Cookie name field, build it using Constant variable that contains the Project API Key, e.g. `ph_` + `<your Constant variable containing the Project API Key>` + _`posthog` .
Example: `ph_{{CONST - Posthog Project API Key}}_posthog`
9. Select URI-encode cookie and save the Cookie variable.

![Image.png](https://res.craft.do/user/full/943950fe-7c34-f4b6-4c10-6b6a92590926/doc/3339FFD8-0290-4458-8227-BA4C0F2CCB7B/10CB5655-5FBA-4EEF-8BCC-3DB05579E790_2/RtK39cPuxyjdKrie3jra30DFmE3T15xG5eOFDCZy73Uz/Image.png)

10. Then, still in GTM, add the new PostHog Cookie variable to GA4 configuration tag with the **key** value `posthog_cookies`.
11. Enable 'Send to server container' in the GA4 configuration tag and paste in your Google Tag Manager Server Side transportation URL. Learn more about setting up Google Tag Manager Server Side. Your GA4 configuration tag should look something like this:

![Image.png](https://res.craft.do/user/full/943950fe-7c34-f4b6-4c10-6b6a92590926/doc/3339FFD8-0290-4458-8227-BA4C0F2CCB7B/9469DEDA-93E4-4261-9C46-5EF4ED3B9521_2/lPYEiW63tQIsT30VYHRfNlmmmP46EE2MgfpJOpXXc4kz/Image.png)

## GTM Server-side preparations:

1. Implement Server Side PostHog tag.
2. Create a tag for events (don't forget the `page_view` event) you want to track.
3. Use the triggers caught by the GA4 client and match them with your PostHog tags.
4. Add parameters sent with the Event Data and exposed by the GA4 client.

## PostHog preparations:

1. **[OPTIONAL]** In Browse Apps, search for and enable the [User Agent Populator](https://posthog.com/docs/apps/user-agent-populator) app. This app will help parse the `$useragent` parameter to individual values. If you do not enable it, all calls will receive the `useragent` parameter, but as a single string.
2. **[OPTIONAL]** In Browse Apps, search for and enable the [Filter Out Plugin](https://posthog.com/docs/apps/filter-out) app. This app will help filter out events. The goal with this app is to filter out any events with the `$lib` parameter set to `web`, in other words, it will not accept client-side events caused by the `posthog.init` method in the base script. In the app, upload the JSON schema as shown below:

```json
[{  
  "property": "lib",  
  "type": "string",  
  "operator": "is",  
  "value": "web"
}]
```

## Caveats

- **[MAJOR]** To send an event to PostHog using their API a `distinct_id` **must be set** on all calls. However, this value should be persistent to allow for proper identity stitching. The call will be rejected if no `distinct_id` is present, even though a `device_id` and `session_id` are present. The challenge is what to set as the `distinct_id` value when no unique user id has been set. In most cases, like Google Analytics and Segment, you can set a device-linked id, often referred to as a `client_id` in Google or `anonymous_id` in Segment. However, when you set a `distinct_id` in PostHog it is permanent. I have reached out to PostHog about this and will monitor any developments. For now, when tracking using the Server Side template, the GA4 `client_id` will always be set as the `distinct_id` for now.

## Feature Backlog:

- Super Properties
- Feature Flags
- Alias
- Group Analytics

## Resources

- [PostHog Javascript SDK](https://posthog.com/docs/integrate/client/js)


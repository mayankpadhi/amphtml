<!---
Copyright 2016 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Custom (experimental)

Custom does not represent a specific network. Rather, it provides a way for 
a site to display simple ads on a self-service basis. You must provide
your own ad server to deliver the ads in json format as shown below.

Each ad must contain a [mustache](https://github.com/ampproject/amphtml/blob/master/extensions/amp-mustache/amp-mustache.md)
template.

Each ad must contain the URL that will be used to fetch data from the server.

Usually, there will be multiple ads on a page. The best way of dealing with this
is to give all the ads the same ad server URL and give each ad a different slot id:
this will result in a single call to the ad server.

An alternative is to use a different URL for each ad, according to some format
understood by the ad server(s) which you are calling.

## Examples

### Single ad with no slot specified

```html
<amp-ad width=300 height=250
    type="custom"
    data-url="https://mysite/my-ad-server">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="200" width="200" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<!-- The ad server will be called with the URL https://mysite/my-ad-server -->
```

### Two ads with different slots

```html
<amp-ad width=300 height=250
    type="custom"
    data-url="https://mysite/my-ad-server?someparam=somevalue"
    data-slot="1">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="300" width="250" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<amp-ad width=400 height=300
    type="custom"
    data-url="https://mysite/my-ad-server?someparam=somevalue"
    data-slot="2">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="400" width="300" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<!-- The ad server will be called with the URL https://mysite/my-ad-server?someparam=somevalue&ampslots=1,2 -->
```

### Ads from different ad servers
```html
<amp-ad width=300 height=250
    type="custom"
    data-url="https://mysite/my-ad-server"
    data-slot="slot-name-a">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="300" width="250" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<amp-ad width=400 height=300
    type="custom"
    data-url="https://mysite/my-ad-server"
    data-slot="slot-name-b">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="400" width="300" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<amp-ad width=300 height=250
    type="custom"
    data-url="https://my-other-site/my-other-ad-server"
    data-slot="123">
    <template type="amp-mustache" id="amp-template-id">
      <a href="{{href}}">
        <amp-img layout='fixed' height="300" width="250" src="{{src}}" data-info="{{info}}"></amp-img>
      </a>
    </template>
</amp-ad>
<!-- Two ad server calls will be made: -->
<!-- The first:  https://mysite/my-ad-server?ampslots=slot-name-a,slot-name-b -->
<!-- The second: https://my-other-site/my-other-ad-server?ampslots=123 -->
```

## Supported parameters

### data-url (mandatory)

This must be starting with `https://`, and it must be the address of an ad
server returning json in the format defined below.

### data-slot (optional)

On the assumption that most pages have multiple ad slots, this is passed to the
ad server to tell it which slot is being fetched. This can be any alphanumeric string.

If you have only a single ad for a given value of `data-url`, it's OK not to bother with
the slot id. However, do not use two ads for the same `data-url` where one has a slot id
specified and the other does not.

## Ad server

The ad server will be called once for each value of `data-url` on the page: for the vast 
majority of applications, all your ads will be from a single server so it will be
called only once.

A parameter like `?ampslots=1,2` will be appended to the URL specified by `data-url` in order
to specify the slots being fetched. See the examples above for details.

The ad server should return a json object containing a record for each slot in the request, keyed by the
slot id in `data-slot`. The record format is defined by your template. For the examples above,
the record contains three fields:

* src - string to go into the source parameter of the image to be displayed. This can be a 
web reference (in which case it must be `https:` or a `data:` URI including the base64-encoded image.
* href - URL to which the user is to be directed when he clicks on the ad
* info - A string with additional info about the ad that was served, mmaybe for use with analytics

Here is an example response, assuming two slots named simply 1 and 2:

```json
{
    "1": {
        "src":"https:\/\/my-ad-server.com\/my-advertisement.gif",
        "href":"https:\/\/bachtrack.com",
        "info":"Info1"
    },
    "2": {
        "src":"data:image/gif;base64,R0lGODlhyAAiALM...DfD0QAADs=",
        "href":"http:\/\/onestoparts.com",
        "info":"Info2"
    }
}
```
If no slot was specified, the server returns a single template rather than an array.

```json
{
    "src":"https:\/\/my-ad-server.com\/my-advertisement.gif",
    "href":"https:\/\/bachtrack.com",
    "info":"Info1"
}
```
## To do

Add support for json variables - and perhaps other variable substitutions in the way amp-list does

Give some advice for how to use amp-analytics

Do some proper support for different layouts - right now, there's strange behaviour if you use responsive
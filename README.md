# Play 2.0.x javascript serving to iOS bug?

This is a test page for what I believe is a bug in play 2x.

## The problem:

When serving javascripts to iOS, somehow the scripts get corrupted ramdomly, with
headers appearing part-way through scripts. Of course this causes parse errors and
the scripts don't work.

With more than 5 largish scripts like jQuery, backbone, underscore etc, this happens
on every page load. With just a few scripts it doesn't happen at all.

## Steps to reproduce:

- Run this app on a desktop machine

- Using a real iPad or iPhone, enable mobile safari's debug console in
  Settings &gt; Safari &gt; Advanced page.

- Visit this the app on the iPad using mobile safari.

- You should see a few javascript parse errors logged in the debug console.

- Clear the mobile device's cache manually in Settings &gt; Safari

- Visit the page again. Note that the number of errors changes and the line numbers
  are different each time.

## Debugging:

- If you install the firebug lite bookmarklet on your mobile device, you can browse to the
  script file / line number of the parse error and you'll see that somehow the HTTP headers
  and (and sometimes body) of a different javascript file have been dumped part-way thgough
  it:

          foo = 17;

          HTTP/1.1 200 OK
          Content-Length: 7218
          Content-Type: application/javascript
          Cache-control: no-cache

          foo = 17;

- In my tests with real javascript files what's actually happening is that script streams
  are being mixed up. Halfway through one script, another will be dumped, including its
  headers.

- The liklihood of this happening is tied to the number of scripts requests, or probably
  actually the amount of data in scripts requested. When the scripts contain a few lines
  then it doesn't happen until you have aroudn 20 scripts being requested. With larger
  scripts, (like jquery, backbone etc) it happens with as few as 7.

## Common denominators:

- this only seems to happen with play. If I serve the same page and same scripts through node.js
  to the same mobile devices, it doesn't happen.

- this doesn't seem to affect desktop browsers or the new chrome iOS browser.

- if I set up a small node.js proxy, and tell my iPad to use it as an HTTP proxy, then for some
  reason when visiting this play app, the problem goes away.

- It reliably breaks on my iPad 3 and usually breaks on iPhone 4.

Is play somehow corrupting the streams in a subtle way that every other browser (and a node proxy) don't mind but Mobile Safari does?


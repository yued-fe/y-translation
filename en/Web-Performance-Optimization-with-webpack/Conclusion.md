
# Decrease Front-end Size

<section style="display: inline-flex;margin: 16px 32px 16px 0;" itemprop="author" itemscope="" itemtype="http://schema.org/Person">
    <img style="border-radius: 100%;min-width: 64px;height: 64px;margin: 0 16px 0 0;float: left;" itemprop="image" src="https://developers.google.com/web/images/contributors/iamakulov.jpg">
    
  <section style="display: block;">
    <div>
      <strong>By</strong>
      <span itemprop="name">
        <a href="https://developers.google.com/web/resources/contributors/addyosmani">
          <span itemprop="givenName">Ivan</span>
          <span itemprop="familyName">Akulov</span>
        </a>
      </span>
    </div>
    <div style="font-size: smaller;word-break: break-word;">
      Contracting software engineer · Blogging, open source, performance, UX
    </div>
  </section>
</section>

Summing up:

* **Cut unnecessary bytes.** Compress everything, strip unused code, be wise when adding dependencies 
* **Split code by routes.** Load only what’s really necessary right now and lazy-load other stuff later  
* **Cache code.** Some parts of your app are updated less often than other ones. Separate these parts into files so that they are only re-downloaded when necessary  
* **Keep track of the size.** Use tools like [webpack-dashboard](https://github.com/FormidableLabs/webpack-dashboard/) and [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) to stay aware how large is your app. Take a fresh look at your app’s performance at whole every few months

Webpack is not the only tool that could help you make an app faster. Consider making your application [a Progressive Web App](https://developers.google.com/web/progressive-web-apps/) for even better experience and use automated profiling tools like [Lighthouse](https://developers.google.com/web/tools/lighthouse/) to get improvement suggestions.

Don’t forget to read [webpack docs](https://webpack.js.org/guides/) – they have plenty of other useful information.

And make sure to play [with the training app](https://github.com/GoogleChromeLabs/webpack-training-project)!
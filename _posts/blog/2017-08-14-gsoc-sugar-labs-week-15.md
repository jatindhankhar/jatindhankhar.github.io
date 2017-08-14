---
title: GSOC Sugar Labs - Week 15
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 15'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 15 '

---
This is the fifteenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I worked added the lightbox styled screenshot carousel. Completed the i18n by generating .pot files using `flask-babel` and started the testing. This week was less productive than other weeks.

#### Screenshot Carousel
Earlier versions of screenshots carousel occupied a big part of screen. This week we introduced lightbox styled screenshot carousel. Most of the classes and implementation were [taken from here](https://bootsnipp.com/snippets/featured/bootstrap-lightbox). I had to remove some `modal` css definitions from `airspace.css  ` since they were overriding the design and causing weird UI issues.

{% highlight html %}
<style>
          /* Modified form of https://bootsnipp.com/snippets/featured/bootstrap-lightbox */

          #lightbox .modal-content {
            display: inline-block;
            text-align: center;
          }

          a.thumbnail:hover {
            border-color: transparent;
          }

          #lightbox .close {
            opacity: 1;
            color: rgb(255, 255, 255);
            background-color: rgb(25, 25, 25);
            padding: 5px 8px;
            border-radius: 30px;
            border: 2px solid rgb(255, 255, 255);
            position: absolute;
            top: -15px;
            right: -55px;

            z-index: 1032;
          }
        </style>


        <div id="lightbox" class="modal fade" tabindex="-1" role="dialog" aria-labelledby="myLargeModalLabel" aria-hidden="true">
          <div class="modal-dialog">
            <button type="button" class="close hidden" data-dismiss="modal" aria-hidden="true">×</button>
            <div class="modal-content">
              <div class="modal-body">
                <img src="" alt="" />
              </div>
            </div>
          </div>
        </div>

        <script>
          $(document).ready(function () {
            var $lightbox = $('#lightbox');

            $('[data-target="#lightbox"]').on('click', function (event) {
              var $img = $(this).find('img'),
                src = $img.attr('src'),
                alt = $img.attr('alt'),
                css = {
                  'maxWidth': $(window).width() - 100,
                  'maxHeight': $(window).height() - 100
                };

              $lightbox.find('.close').addClass('hidden');
              $lightbox.find('img').attr('src', src);
              $lightbox.find('img').attr('alt', alt);
              $lightbox.find('img').css(css);
            });

            $lightbox.on('shown.bs.modal', function (e) {
              var $img = $lightbox.find('img');

              $lightbox.find('.modal-dialog').css({
                'width': $img.width()
              });
              $lightbox.find('.close').removeClass('hidden');
            });
          });
        </script>
    {% endhighlight %}

Here is the end result <br> 


<img src="/images/gsoc-week-15/screenshot_carousel.png" alt="Screenshot Carousel"> 


and when user clicks on the image, it will enlarge


<img src="/images/gsoc-week-15/screenshot_carousel_zoom.png" alt="Screenshot Carousel Zoom">


####  i18n - Adding Pot Files
This was accomplished by using `flask-babel`'s  `_` or `gettext` function to substitute the template text with the language. 
To generate the `.pot` file, following command was used `pybabel extract -F babel.cfg -o messages.pot .` 
`babel.cfg` contains the patterns and files to look for `_` or `gettext` function and adds it to .pot for translation. 

Here is the configuration used for aslo-v3 

{% highlight python %}
[python: **.py]
[jinja2: **/templates/**.html]
extensions=jinja2.ext.autoescape,jinja2.ext.with_
{% endhighlight %}

Also a proposal to include aslo-v3 in the [SugarLabs Translation Platform](https://translate.sugarlabs.org/) was sent to [Chris Leonard (cjl)](https://wiki.sugarlabs.org/go/User:Cjl) and he replied affirmatively :smile: and we may soon have new place for aslo-v3 translation.


#### Testing
Testing at the moment is not robust and not confidence worthy but I added some initial tests. For testing we are currently using [`pytest`](https://docs.pytest.org/en/latest/).

I added some unit testing code for `i18n` api module and some integration tests.

Here are the integration tests, they uses `setup` and `teardown` to allocate resources for running a class of tests and deleting the resources. 

{% highlight python %}
class TestSimple(object):

    @classmethod
    def setup_class(cls):
        # Most hackish way to run the server
        subprocess.Popen(['./start.sh'], shell=True)
        # Allow grace of 5 seconds, server bootup time
        time.sleep(5)
        print("Setup. Starting the Server")

    def test_if_app_starts(self):
        app = init_app()
        assert(app)

    def test_app_should_respond_ok_when_index_page_is_opened(self):
        res = requests.get("http://localhost:5000")
        assert res.status_code == 200

    def test_app_should_redirect_request_without_lang_preference(self):
        res = requests.get("http://localhost:5000", allow_redirects=False)
        # zpytest.
        assert res.status_code == 302 and (
            "localhost:5000/en/" in res.headers['Location'])

    @classmethod
    def teardown_class(cls):
        os.system('killall honcho')
        print("Teardown. Stopping the server")
{% endhighlight %}


To run the tests I modified the `.travis.yml` to run the tests along with pep8 style check `flake8 && pytest tests/`

## Goals for Next Week
This week I intend to focus on writing more sensible tests and writing documentation (maybe using `sphinx`), migrate database of aslo to aslo-v3 and move the server to new VM since my old VM died (DigitalOcean credirs expired :sad:).
If you find any typos,mistakes or any other inconsistencies, let me know and I'll fix them.
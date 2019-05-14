---
layout: post
title:  "Running long CKAN imports"
date:   2019-05-14 15:00:00 -0400
categories: CKAN imports
---

One of my current [CKAN](https://github.com/ckan/ckan/) setups is a pretty
basic Source install running on Ubuntu 16.04 LTS. For a few different reasons
we set up the server to be as small as possible which for the most part is
great but it does have it's limitations.

The VM is on Azure and is a tiny A1 Basic General purpose machine with i
VCPUs and 1.75 GB of RAM. Good news, its cheap. Also, depending what you're
doing with CKAN it seems to handle it relatively well.

Where it starts to stumble is when doing some imports of datasets.

For importing in this case I wrote a python script that loops through a CSV
export from another system. I then wrangle the data into my schema. During
this process I have to clean many of the values to be more appropriate and
clean. I also have to use Selenium to scrape another site to get some
additional information for some of the records.

It turns out it was the web scraping that really killed my VM. It would start
nice, run for awhile then start to slow down and the CPU would continually increase to
around 90%. Eventually I would kill the process and remove some records from
the CSV that were already imported and start again but this sucked.

I ended up resizing the VM to an A2 Basic General Purpose VM with 2 VCPUs and
3.5 GB or RAM. I also added a couple tweaks to my import scipt in the Selenium
department.

For Selenium I added some arguments to ensure it was a bit more streamlined.

{% highlight python %}
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--disable-dev-shm-usage')

    # newly added
    prefs = {'profile.managed_default_content_settings.images':2}
    chrome_options.add_experimental_option("prefs", prefs)

    driver = webdriver.Chrome(chrome_options=chrome_options)
{% endhighlight %}

I went from an implicit wait to an explicit wait (should have had this to
begin with.)

And finally, I added a very small `time.sleep(0.2)` just before I call a
function that handles creating and dealing with the driver.

This combination seems to help keep my VM relatively small but handle a larger
amount of work without killing my VMs CPU.

## Additional Notes

Yes, there are other ways to do imports such as the ckanapi and the harvester
extension but I've found that when I need to wrangle the data to align with my
schema it's easier to write a script that handles it.

---
layout: post
title: "Learning Python Scrapy with example project"
date: "2017-08-14 15:46:39 +0300"
permalink: /:title
---

[Scrapy][scrapy] is a cool tool to extract data (scrape) from websites. To learn it I decided to create a crawler to crawl Finnish namedays from [www.nimipaivat.fi][nameday].

[**Here is the source code for the project (nimipäivä JSON)**][sources]

## Items, crawlers/spiders and pipelines

I've written all my items (just one) to `items.py`

```python
import scrapy

class NamedayItem(scrapy.Item):
    day = scrapy.Field()
    month = scrapy.Field()
    official_names = scrapy.Field()
    swedish_names = scrapy.Field()
    same_names = scrapy.Field()
    orthodox_names = scrapy.Field()
    unofficial_names = scrapy.Field()
```

Spider/crawler emits these items and pipeline will save the items to memory.

```python
class JsonPipeline(object):

    my_items = []

    def close_spider(self, spider):
        with open('items.json', 'wb') as f:
            f.write(json.dumps(self.my_items))
            f.write("\n")

    def process_item(self, item, spider):
        self.my_items.append(dict(item))
        return item
```

`process_item` is called for each item. I'll put the items to list and write the list to JSON file in the end. This approach is problematic if the crawler dies before the end, nothing will be written. It is good enough approach for now.

Parse method is the method which extracts the data from the server, populates the item and emits the item to pipeline. I've commented out some of the namedays, because the list belongs to someone else (see [copyrights][copyrights]).

This particular crawler writes the extracted data to `items.json` file.

```python
    def parse(self, response):

        official_names = []
        swedish_names = []
        same_names = []
        orthodox_names = []
        unofficial_names = []

        date = response.xpath("/html/body/div/div/div/h1/text()").extract_first()
        ps = response.xpath("/html/body/div[@class='container']/div[@class='row']/div[@class='col-md-6']/p")
        for p in ps:
            if "Nimi" in p.extract():
                official_names = p.xpath("strong/a/text()").extract()
            elif "Ruotsinkieli" in p.extract():
                swedish_names = p.xpath("strong/a/text()").extract()
            elif "Saamenkieli" in p.extract():
                same_names = p.xpath("strong/a/text()").extract()
            elif "Ortodoksista" in p.extract():
                orthodox_names = p.xpath("strong/a/text()").extract()
            elif "virallista" in p.extract():
                unofficial_names = p.xpath("strong/a/text()").extract()

        # Extract day and month from date string
        extracted_date = date_pattern.findall(date)

        # Populate the item
        item = NamedayItem()
        item['day'] = extracted_date[0]
        item['month'] = extracted_date[1]
        # Uncomment these lines to make this crawler crawl forbidden names
        # item['official_names'] = official_names
        # item['swedish_names'] = swedish_names
        # item['same_names'] = same_names
        item['orthodox_names'] = orthodox_names
        item['unofficial_names'] = unofficial_names

        # Return item to pipeline
        return item
```

## How to run Scrapy crawler

`scrapy crawl nameday`

The crawler/spider name comes from the filename [nameday.py][nameday-py].

## Other important things

Default Scrapy User-Agent is forbidden in many websites these days so make sure to change the User-Agent in `settings.py`.


[scrapy]: https://scrapy.org/
[nameday]: http://www.nimipaivat.fi/
[sources]: https://github.com/spedepekka/finnish-namedays
[nameday-py]: https://github.com/spedepekka/finnish-namedays/blob/master/extractor/spiders/nameday.py
[copyrights]: http://www.nimipaivat.fi/tekijanoikeudet.php

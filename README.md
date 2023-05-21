# learning deneb (or is it vega/vega-lite)
Over the next couple of months I will use this repo to share my learning about using deneb for creating data visualizations inside Power BI. If you are not familiar with deneb you should start here: https://deneb-viz.github.io/

I consider deneb being a very powerful custom data visual available for Power BI. deneb does not provide a certain data visualization to solve a data visualization task instead it provides a framework that allows us creating almost any data visualization we need to communicate insights with the users of our Power BI reports. During the last years I encountered various requirements for specific data visualizations that have not been met by the default visuals or the custom visuals available. Sometimes these requirements could have been described as minor additions to the way information is presented, sometimes these requirements only could be met by creating R or Python script visauls. Sometimes I considered creating a new custom visual. All of these custom approaches have their downsides, no matter how powerful packages like ggplot2 or seaborn are, the data visuals created using this are lacking interactivity the users of Power BI are used to. Creating tailor made custom visuals requires additional tools like Visual Studio Code for developing the custom visual. From my point of view the amazing freedom that comes with creating my own custom visuat that fits the needs exactly comes with a price tag, maintenance and the integration of additional functionality, meaning new features. Next to that, I do not know that many business users that are familiar with the required developent skills.

I consider deneb different, because it provides a framework where I, the report author can focus on the data visualization task without the need for additional tools. When I say deneb provides a framework where almost any data visualization can be created, this of course also states that there are certain tasks that can not be solved. This is true for two reasons. The first reason of course is simple, it's simply because of my lack of knowledge, but this is what this repo is all about sharing my learning experience. The second reason is a more complex one. At the current moment deneb embeds two javascript libraries used for data visualization, vega-lite and vega. There are certain data visualization task that can not be solved with the above mentioned libraries alone, e.g. the mapping of data to spatial aesthetics. But I'm happy about these kind of limitations as there certain capabilities of a custom visual require communicating with third party services, this does not necessarily mean that data is not safe, but this prevents the visual from becoming a certified. From my personal perspective I consider being certified more important than being able to talk with third party services.

deneb allows to describe how the data at hand should be visualized by providing a specifiaction, this specifiaction is using json notification. Then this specifiaction is used for rendering a data visual, that out the box supports cross filter and tooltip capabilities (thanks to the inventer of the deneb custom visual). If you think this is simple, you are correct. Unfortunately, the vega-lite and the vega library provide a lot of means to manipulate the available data, create new data using transforms and expressions. Because, it is required to provide the "formula" in a syntax I'm not familiar with, I'm struggling. I thing what Alberto Ferrari and Marco Russo once have said about DAX is also valid for using vega-lite and vega: Using vega-lite or vega is simple, but not easy.

As I'm convinced that deneb will play a great part when a custom visual is required, I will learn how use it.

Throughout, the next months I will use this repo to share my findings, my projects - the data visualization tasks I'm currently working on.

The next sections will desribe the topology of a specifiation, its sections like mark, encoding, layers, and all the details that have not been from the beginning. This is not about how to use deneb inside Power BI, this is described in very great detail on the official site.
# View composition - one of the building blocks of vega/vega-lite
When I learn something new I always try to understand the building blocks, the blocks that are forming the foundation. Like a subscripiton, a resource, or service principal in Azure. Like the evaluation context, extended tables, or filter propagation in DAX. Like the concept of a lakehouse in modern data warehouseing I do not know how often I wake up in the morning thinking that these two phrases do not fit together. You might not agree with what I consider being building blocks so far, then consider the things in this section just being simple notes I stare at, when I have to understand why something is not working as expected.
On my journey learning vega / vega-lite I consider a view one of these building blocks. The next sections describe how I understand a view and how I created this tiny table. do not be afraid, the following sections are not about the creation of an ordinary table (it's not that ordinary, though). I will use the fancy stacked bar chart (to be honest of my favorite data visualization types).

![the table - more than the obvious](https://github.com/tomatminceddata/learningdeneb/assets/29025119/cccb3bf7-d4ab-4519-8160-de1d29aa60e9)

The image above was inspired by this question on stackoverflow: https://stackoverflow.com/questions/75975968/table-visual-with-customized-grid-layout-in-deneb-vega-lite/76277789?noredirect=1#comment134512446_76277789

Here you can have a look at the [vega-lite specification](https://vega.github.io/editor/#/gist/db3a824af2ef0b6e6204d1a3ee56d6f4/TableAndVerticalConcat.json)
## What is a view - and why it's important
I consider the vega/vega-lite view being a window that allows me to see a data visualization. When I'm looking at a data visaulization I'm often one of two personas, either a reader who wants to learn from the visualization and decide about the next actions, or I'm a creator, meaning I create a data viz that others will read. When I'm a creator and looking at a visualization that I did not created myself I often ask myself how the visualization was created. Sometimes I consider this a training or a mental excercise - decomposing a data visualization into the ingredients I know, meaning vega-lite or vega.
The most simple view is a single-view data visualization.
## The single-view data visualization
Thinking in vega/vega-lite - the specification of a single-view contains only one mark without any view composing functions like facet, repeat, or hconcat. If you have no idea what the former functions are good for, bear with me, I will explain these functions (at least two of them) in one of the following sections.
The following gist provides some simple data and a spec. The data and the spec become more "complex" throughout the following sections. Please be aware that the specs are not representing best practice advise in data visualization. The following sections are covering only the technical aspects of visually composing data visualizations.
[The gist points you to a specification of a single view stacked bar chart:](https://vega.github.io/editor/#/url/vega-lite/N4IgJghgLhIFygG4QDYFcCmBneBtUATgPYDuA5sWgA7wgCSIANCFhmQLYYB2UtAwkxDIU8AEwAGSVIC+jQqQpFqtBs1YduvOCABCg4fADsUmXJDFylGtrqqWbTj1oBBfangAWE5NnzLS63o7dUctXTcROABGb3FfcwUrFWCHTVoAEQixWOkAXVkQdggCAGt4UCgATyoMWgAjYpAC7gBjIjAASy4ycpBK3oAzDowUMFoLRWVmKpraLiJ2LvdmCAAPDpwEECgOqBRauC40FBRpAtXB4dHaA2nqg5AARzQIHl3oDsRalfXNit39vAjiczsw2igiARLiMxtoQmk7rNtPNFlx3GdpEA)
And of course the image:

<img width="184" alt="view composition - single view" src="https://github.com/tomatminceddata/learningdeneb/assets/29025119/01f5f438-cd2e-4944-b31d-9d43316c0c69">

If you have a closer look at the specification, you will see that there is nothing special about it, it's simple, it's this:

```
{
"data": { ... },
"mark": { ... },
"encoding": { ... }
}
```
When you are using analytical tools that come with build-in data visualization capabilities like I do, I use Power BI, then it seems very easy to show data labels, not only for the segments but also the total value for the entire stack.
This is not that easy when using vega/vega-lite the reason for this, is also simple. There is no UI where we can turn on the setting "data labels." Instead we have to start thinking in compositions of marks forming a data visaualization. For this reason we must understand what a mark is. Here is my simple definition of a mark:

A mark is a visual representation of data in the form of a geometric object, placed on a 2-dimensional plane.

The most commonly used marks are rectangles and points. Using libraries like vega/vega-lite makes visualizing bar charts like the one above easy as we do not have to consider how to draw a rectangle and even more importantly we do not have to consider where exactly we want to have the rectangle drawn.
But we will also realize that we have to take care about showing the data labels by ourselves, as there is no UI that presents us with myriads of properties that we can use. Instead we have to become aware that a data label is a second mark of type text, that is using the same set of data. A second mark that comes with its own encodings but is "stacked" on top of the mark that defines the bars. A little reminder encodings are the mapping of data to visual properties of a mark. The mark bar for example has visual properties like width, height, color, and some more that are not as obvious as the former ones. I consider it very helpful to have a good understanding of the available marks and what data visualizations can be created using each of them, as very often data visualizations that are aestethically very appealing are a composition of multiple marks.
## Multi-layer views
Adding data labels to a "simple" bar chart transforms the single-view data visualization into a multi-layered data visualization.

![View composition](https://github.com/tomatminceddata/learningdeneb/assets/29025119/8a215f3b-ea6e-4ba4-804a-0dcb1bcd8088)

The above image shows (hopefully) how two layers, to be precise three layers are stacked on top of each. These layers are
1. the bar chart (mark - type: bar)
2. the data labels of the segments (mark - type: text)
3. the data labels of the stack (mark - type: text)

Creating multi-layer data visualizations is simple just wrap the function layer around multiple groups of marks and encodings, each of these mark/encoding combinations is representing a single view, like so:

```
{
"data": { ... },
"layer: [
  {
    "mark": { ... },
    "encoding": { ... }
  },
  {
    ... layer 2 ...
  },
  {
    ... layer n ...
  }
}
```
The creation of the above data visualization requires some more effort, but once understood it is easy, here is [the gist to the stacked bar chart with data labels](https://vega.github.io/editor/#/url/vega-lite/N4IgJghgLhIFygG4QDYFcCmBneBtUATgPYDuA5sWgA7wgCSIANCFhmQLYYB2UtAwkxDIU8AEwAGSVIC+jQqQpFqtBs1YduvOCABCg4fADsUmXJDFylGtrqqWbTj1oBBfangAWE5NnzLS63o7dUctXTcROABGb3FfcwUrFWCHTVoAEQixWOkAXVkQKAIILiwAMyICdjxQEgBLLjBSGpAiQIsAfS40dgAjDAJBCBxtCzpGjAAPHQBPAHErZ0aAZVSnPOZFal6ZvAT-ZTU13nyzACsiBogyCjZoDBayuowUMFoDZjbaLB6hkfsNDxlr8NiAtlQdnsLODBCE0qdQBcrjcCHcoA84PgQE8Xm9tB9WoEftVmMNaODgdVQeDIZj9jCEdi6ih0YNtJAoD1cAByMYTabzRYrY7c3IAXglURA8XqjWamNAX1GpC6PX6g1J-z5YCmswWAWluU2VlpuHpSSNLEqWixONe32OgkqOrZIGGAGNuGAGmRDTKGk0SC0lSxfsw7XiAaFKX9aAQ0FwuD7KbNVoDeNSTbs6dCLWprY9nvblSRxjqBfrDq0CC7aB6vT6-WZ3ah3WgUPdaByubyE0muGQUzM06FRQACAC0Y+77B5cKBPVFse0kxHmh0wweoI7MwGLXYEAIAGt4KAoDMqBiQL1D9LmNx3URvQPTyBJq+I+93Mxz5faABHNASigOoYBAxAHlJSY6hGM9QJQDFuhQFBpAKbNQE-EsYR-C8ry4Ih2CuEQoJg18QKgBD4CQlCCkfFBKg-ItI3nXgcL-bR8MIrhvxABCyC9Mj4MQ9sUE+AhnicbReiIKAoAI6VUPiA9jzI3DaHRSZWJAOiGKkjt3RPUkUDqMguFoBCyi071XQICjBDAd84FEAoHyfRsEDfRjcVoVdjg3VhBF-K9AOA0DoDqCC7xAdCmW8rCkjYvCCKIqKNK0DCmK-YjCjU7QQp4MLwMg7FKgPMIADpxBwRSzGUk8PKC9Spi01ATLM7QLKsxyAFYXK4R9n19DzHIyuKwSsGNEoAoCCrAiKt2YGLMPNA0po45LuJEAo0q84txoCSacvYkB8vI8LIvDUroFoSrqtQy1USwIgUEijysBbSiPJ010AwwS8JnWVCgA).

If you are wondering why so many data transformations needed to create data labels - this is explained in greater detail [here](https://github.com/tomatminceddata/learningdeneb/edit/main/README.md#the-transform):

The function ```layer``` stacks multiple single-views on top of each other.

Please be aware that this is just a very simple example of multi-layered data visualizations. If you combine a bars with ticks you can create a bullet chart with ease, see [this gist](https://vega.github.io/editor/#/url/vega-lite/N4IgJghgLhIFygG4QDYFcCmBneBtUA5gE4D2aADvCAJIgA0IscIAwvSAEbwBMADPwIZQe3AKwD+AXzqFSFKrQZNW7LnADsE-kJHiJ02WUrNFjKmwZqALFt4643MVoMhiRhe2UARVfACMtvaOegIubvImnlQ+ljyBIMIOTvoyrnLGNFHMMZzwAGzxicHOqeEZ1KbKAKK+cAAchf51IVKl6QqVVDWxcDZa9n7NJYYRNJ3M3bkajXCDLbySALrSCUQQAHZYAGYkRAC2eKAA7gCW62AkR4cgJMa4IKRHAPrraHscGEQgi0o4cPePajnDAADwAQgBPADi6QAgucAMoYAh7DDrYTLH5pIwcCF4bHyJTfFwAKxIZwgBGIyOgGGuWxOGBQYHxXCxt3xWDe31+nORqPRCO5mIYZVx+Ngovay1SZIpVKINKgdP+oAZTJZ-1y7LuIC5ByxED+9zKQoNSylOLxWrKxNSDJQyq+zEgUDeuAA5IDgeDoXDEfy0VAPYsALzhvwgFync6Xa4crWPF5vD5fQ3Gh6XIFgUGQmHuEUE8jim3ShhYXaJfAgdXMqiSm5EHPOxhYADGaLAZwIUZlxzOFyuqpuGX17Frmr1gcF3N5zCIaHW627ZshSJRQd7looJZNZb1lfpjLr86zPrz7QYu2b9fbne7vZcbdQbbQKFpVFd7q9i+X6wIq4QuuArBosAAEAC0YFfnsnpYNOUBmiGnh-CAILAUGYJGnSFogG2JDrAyPYIHhBHPokoBYOQEBtg+cALCsAAW+HrOR1xQCcUAoCqCSMScOAMHsEBEAA1vAoBQBC5A8cqILCAwYAgv4CwMGi+Fdv+4kgNaarHpOtpCFJPHrCQewUignggvxWkcVxKqvCgKCSCsskUTWelUKas41rsQmJCAAB03A4M5LjvhCnzsZx3FUFAjHQOwQmiTZRlUBwwnsIxGAnAQjFuel5zwLwAUAJzOaprEkBpxGgDp7kap5l4JKlzAmWZ6yoJZ1lwA5TkMEpJETlQ8EbjOByGdJVAAI5oBstnQCciB0koVl-BJ0X2W+KAMDs+wJcwQUhSs+EoLsWlDcwDbcQQnY2Rt8C9VeRCMuiaUkFAUCmVGoWpElYkkZJk3MK57AnWdzBHHxyrsF2LaOl8SgoDl6xUNxWzyeAA3cCsalVXRtXnR5zAGc1QMgG15lRv1hMNcw6EIVh8HsIDPEzXNnELUtVMJKCbkXVOo2Id5u1+VQh3fS4f0pWTIMMGDLYcO+bZiQpJxw1ACOMEjBAo8waMY4pPA45V1VaXV-MkyzVAUx1FkrANum00WZrMy1IBs+iHMcVzLm8zTJ7O8Lvn7YFwUS79wn-RJbsccroMkKdCtKyrnAbGAWAnAAXiqADMdi8ScyvrNgfzYxV6n49p-v6U1VutaZlP29XsWu2THvzd7OHOX2CT3cwFaonF3bQScWxbJ8m6CZH0syX7CkDQEcsJ+DnDJ9zuOmyR5tE0WrfGQ3ttdX8vW+3JWnIOgPEBdfEtYoqFYoFzJHXbdJHy1QA4YNJwKvaFQA).

Another example of a multilayered composition is the creation of a lollipop visualization that is often used to visualize differences. This type of datav isualization combines the rule and point mark. A lollipop visualization is used in one of the next sections.

Here you will find many more [examples](https://vega.github.io/vega-lite/examples/#layered-plots).

## Multi-view compositions
Functions like hconcat or voncat combine multiple views (no matter if it's a single-layer or a multi-layer view) into one data visualization, either horizontally (hconcat) or vertically (vconcat). Functions like facet or repeat are creating a trellis of a data visualization. If you are more familiar with Power BI than these two functions are doing what small multiples are doing. The difference between facet and repeat is this:
+ facet - facet filters the dataset by the specified definition
+ repeat - repeat provides the entire dataset to each visual
The next image shows an example of a multi-view composition:

<img width="343" alt="view composition - multi-view" src="https://github.com/tomatminceddata/learningdeneb/assets/29025119/905339c7-90c3-4c7a-80ef-0c9a72d92f27">

The [link to the gist](https://vega.github.io/editor/#/url/vega-lite/N4IgJghgLhIFygG4QDYFcCmBneBtUAZhAMYZQDmATgPZoAO8IAglgJawA0INA7lbQzggAkiC5YM5ALYYAdlEYBhMSGQp4AJgAMO3VzV1KGRPADMu3QF8OhEmX71GLdit4PBIlROlyFQgEIqavAA7BY6+qiGxvAAHOFa1rakFDSOQs6c3NR8aR7CouKSMvJOQajwACwJkSjRJnDm4UkgRCnuTmxZbnmMBV7FvoyBtfAAjDWqUUYNE802rXapAp0uXD0rQv1FPqVCACLl6nDa4bX1mgmWALrWIFCUELJYBNSUUnigPKyyYDmfIGoHl4AH1ZGgpAAjDCUFQQHBCXjCX4YAAe-gAngBxPJMX4AZUGpRuXHckIxeEW7V66xyHR2JQUtwWACtqD8IOQqJJoBgAQRWBgUGBGMEuEDGFgIXCESBvIz8dKSSAyRS4LgqfYadlcisGUNmaA2RyuUZyLz+YLhaKKuKPFKPlx4Yx3IqPsrVZS2lq9TqOoaQMbZJzueaoHz1YQrSKhAYZioJUIHTKXXk3QAFeMevLkr1Lel+3oBgUocOwoSQKAQ3AAciRKPR2NxBKJUBr1wAvF2xiAWt9fv9I4DgTkwRDobCnbL62A0ZicStswJc+rNctHLTdY5ruI3n58K1o5LWwnKLPyyB4aRfj9yL2A-2-jwAYm5dKuAKhTG5a23SnEWgsiyLebqYoSuwKEu9Arhq3rrgwm7+rulD7lGX6MDOc5Nr6bznowV5yGAt73i0xCoMQaAoBaFbQNWdaAcBsjkKBGLgYy7YAAQALQcZWdHyr4brtv+IComxvj+PCfKkeRlHUeAtFSLWroQpxPF8UpNYqVImbGMJU6MMQAAWTzkBgukmMqRnENQshkahIBURiMIAlArBQCgEYgIoaCUEY8gcc5ECTiARkYKw5BGX4ACsWhcFIwUANbwKAUAYnQXmQsFvZcHINlEUxKWiUVn7WrGtr3OlXkAI5oE8bkwG5iB8k6qKsAiqXuZ58DgigKCWHcapoWVha+mlGWMLI1BSBy6ite1RUNd1cC9f1dw2SgbwlUeSYnlw41eVNM3BnNjmSIRi1dRGq3ipQgp7CAkLUFAUDTb2A0tAllDJQglUTUI4aogoXAbVtQiQlRxDJU6KARbIjCeQQwPgKwF4oadYCopodx5dQBV3r9WO-aV35ia2kkSCoB2MLV9XudArDNTlIBDYe6GInS2rU0IR2zczgN+MN35in9NV1fI9NNS1rRvAlfggAAdFoOAfQsX0-alVWMALcKw+Q8NCIjyOY-A0U43ZePEYT23syqabvqLNPiw1DNM4NNsjRsG6Ozz0183cOvEztdsCH++1a0ItMS41jPS687zQIwSsqwNAZOS5Q5hRFUWm3FIDfGAUBGeMET3FdSgmUxGAcdQzWUBx9SsLQWCBRg2XxUli0R9wlHSblFv40VrMkxhnNjd3vMnXCbUIqtdxE0LhmV2ZFlU93Ucu1L08Lb9S3XZRa0g9Qm3lqANk3m5tmLdg8sabWxmmeZMycQAPBxWhHJgGEYCKdxqF-4NIbJVVqAdWXd-ogDoOyUo4gTIQOIKjYg3UPysD6j-eADxMDm3ylbUAw9g5ewQj7EAk8Kjzw9t+B+VdV7hwgRvSWsdmag1PiAc+RFL4G1SjfRgd8axUJXs-a4HE34f1qAA7g6C-6oHERDEgwDU43EsEAA).

The specification slowly becomes more complex :-)
```
{
    "data": { ... },
    "transform": { ... },
    "hconcat": [
      { layer 1 : [ { ... } , { ... } ] },
      { layer n : [ { ... } , { ... } ] }
    ]
}
```
And of course,  the "final" iteration adding the facet operator to the specification. The image:

<img width="421" alt="view composition - multi-view faceted" src="https://github.com/tomatminceddata/learningdeneb/assets/29025119/0835d89e-a40f-45e8-bd38-d4b961ee1dd6">

[The gist](https://vega.github.io/editor/#/url/vega-lite/N4IgJghgLhIFygG4QDYFcCmBneBtUAZhAMYZQDmATgPZoAO8IAglgJawA0INA7lbQzggAkiC5YM5ALYYAdlEYBhMSGQp4AJgAMO3VzV1KGRPADMu3QF8OhEmX71GLdit4PBIlROlyFQgEIqavAA7BY6+qiGxvAAHOFa1rakFDSOQs6c3NR8aR7CouKSMvJOQajwACwJkSjRJnDm4UkgRCnuTmxZbnmMBV7FvoyBtfAAjDWqUUYNE802rXapAp0uXD0rQv1FPqVCACLl6nDa4bX1mgktbfa9QgCqAMquOR1bA7t+IMqjjZMGMzMVwWN2W6RATxeuU2nh2JS+IymxzmZyRFzg8XmyVuMMh61edxEhRA3nhZV+1VRAJifyxi3ahLx2Wh4O2JMGexAiOCJ3+0xpKKsIKWbwhz3xLPyxNJQwOR0uVP5DVOFksAF1rCAoJQILIsARqJQpHhQDxWLIwDkTSBqB5eAB9WRoKQAIwwlBUEBwQl4wgtGAAHv4AJ4AcTyTAtjw5CnVXHcLuDeHpOMcEo6cKGGoWACtqOaIOQqJJoBhrQRWBgUGBGMEuLbGFhnZ7vezPo9m3GQAmk3BcCmwQx071M6Vs6A8wWi0ZyKXy5Xq7WKvWPE3jVwvYx3B3jV2e8nQRnmR1xyBJ7JC8XZ1Ay33CAua0JqSYV43mxvW9vnQAFQF7vKJgeIojsevSnhWKA3h6QiQFAzq4AA5L6-pBmGEZRjGCFqgAvLhYwgC0ZoWlad42naOSOs6boeh+jDIWAgYhuGKz-gIgF9gOR4bI4ariIafj4K0D6NjGKiGgx0EgF6pAWua5AEaeRGWjw1oNkIa4qBWVaPm28I7i2dFoLIshyTuIbRp8CnxgBvb9oeIHcQwvEkvx87aXROR+gxqHMWmNqUBJjDSXIYByQpLTEKgxBoCgc4wdA8FIUZJmyOQZnBhZ8JYQABAAtNlsGJTK8g7lhBlCAGmW+P4XplhFUUxXF4AJVIiFflIOX5YVrUIe1v7GGVtFCMQAAWurkBg-UmF2I3ENQsiRQJoBQKwUAoLey2Bl80WUEY8jZcGGAQDRUkLSNhqMFIrBgGA62eigrDkLIjAyVBBFcKC8CgLwX1Ce5Qj2SsXAjUdgUICAsVuigTAPU9jDrQQChcJDVaRuQd1wFoyMQFD34QDdYVwBMlgk+IdAYMQv2xYd0GCVIx0ANa-VAwbk4wLrHe9IByHNoWpb9Aa-Vpi5PsuWqs7eIAAI5oLqK0wCtiBlhuAasN6y2rRjTooCgH2GvTXwAHQaDgpMgL297-aBQPi2zQiyNQV0XuoKtq8zmu3trKBm3NKAXeDws6cVSO25LDtO2L60TRa7trZ7MW6-5laci61BQFAjsESTLT05QTPgyzdtaltKi+-7IAurFxBMxusPPUICMh6FkmUGtKhgILJyajz1B8-J4Od5bIsgJVMY1RIKiF5LMty6t0CsErXMW39w+OZPEuMOHBbqJqN4Bn4Q86XWoeMDP8hz4ryutPr0CMIbWim6ToC5-ny0b0Ie8h6gj31xDGCI+3TuABWbuC1e6E1AIPFeOl2rryLmfeW89F6amXoHDyko4Fh0dtvLmn8hbCSELArgU9T6y3PgrBeV8DRGlvkIY2j91RPy1B7F6Y1UoYGytQJWlBspQBBtleorBaBYAOkdE6n1wY-QDgQziIEQb43dPAL2ZssDk0puDamijSIv2Zu-bgMU6pcB7n3X6qCZFr1dt6ZRXAoFoOGmwiaU1MGkNnhQxeljY5awTnrGhRsTZZy4GXaCoA5qyRWvNZm2AvjdUQqNcak0Zg5QADzZS0EcTAdEMA1k1GoDJQhK4kCZtnBYOiC56LoPmUoH1WA6yyfAbUmBQG8wgebfBVsLFSVVlYhOmpbEyLiewpxxC9EIIvpQz0XTPHxx1j7agftgkgFCaFcJ9dNpYGiS1WJDiEkDTVNlFJaTah5O4HUnJqBjkFOrlnRhjCgA).

The specification is growing:
```
{
    "data": { ... },
    "transform": { ... },
    "hconcat": [
        {
          "facet": { ... },
          "spec": {
            { layer 1 : [ { ... } , { ... } ] },
            { layer n : [ { ... } , { ... } ] }
          }
        },
        { 
           the lollipop visual
        }        
    ]
} 
```
## View compositions inside Deneb and Power BI
When using a single view or a multi-layer view with Deneb we are spoiled, because the visual adapts when it's resized on the Power BI canvas. Vega/Vega-lite are currently not supporting auto-resize for multi-view data visualizations. From my experience this not an issue, as the width/height can be adapted to the space available very easily. The below snippet shows how to set the width for two horizontally aligned views:
```
{
    "data": { ... },
    "transform": { ... },
    "hconcat": [
        
        {"width": 100,
          ...
        },
        {"width": 200, 
           ...
        }        
    ]
} 
```
## How do I create multi-view data visualizations
When I create Deneb visuals it often starts with a hand drawn sketch from one of my colleagues, when it seems it will become more complex I tend to create a concept layout using Power Point. The next image shows the conceptual layout of the multi-view composition of two horizontally aligned facets of multi-layer views:

<img width="350" alt="view composition - conceptual drawing" src="https://github.com/tomatminceddata/learningdeneb/assets/29025119/5ea69296-d271-4721-8bdc-bc6bdd14463f">

These conceptual drawings help me a lot to stay focused, but I also use these conceptual drawings to communicate with my colleagues.
## Resolve - or how to break the rules
There are a couple of things that must be considered composing multi-layered and/or multi-view data visualizations. It's more than likely that one layer or one view, or maybe both are using the same encoding channel, e.g., color. Vega/vega-lite by default are combining these encodings into one axis, one legend, but sometimes your data visualization task requires something different. This is the moment when the function "resolve" has to weave some magic. Here you will find the official documentation:  https://vega.github.io/vega-lite/docs/resolve.html
I'm pretty sure that I will cover my learnings about resolve here in one of the next updates.

# The specification
# The transform
15 months have been passed between publishing this repository and writing this section, but I learned some new things, forgot some of them, and re-learned them. Now is the time to write these lines, as I do not want to spend time on things I already solved once too often. I realize that I'm still struggling with the syntax of vega/vega-lite. I assume this is because of the time I'm able to spend with deneb, and not because of the complexity.
A vega/vega-lite specification might or might not contain an array of data transformations. Remember the specification of a data visualization for a single view, here a bar chart is depicted:
{
"data": ... ,
"transform": [
{ ... }
],
"mark": {"type": "bar", ... },
"encoding": { ... }
}
To better understand what this article is about. The following data

```"data": {
    "values": [
      {"group": "I", "a": "C", "b": 2000000, "t": 22500000},
      {"group": "I", "a": "C", "b": 7000000, "t": 22500000},
      {"group": "I", "a": "C", "b": 4000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 1000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 2000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 6000000, "t": 22500000},
      {"group": "II", "a": "E", "b": 8000000, "t": 18500000},
      {"group": "II", "a": "E", "b": 4000000, "t": 18500000},
      {"group": "II", "a": "E", "b": 7000000, "t": 18500000}
    ]
```

will be transformed and visualized. The next image shows the data visualization:

<img width="295" alt="transform - the visualization to achieve" src="https://user-images.githubusercontent.com/29025119/236114636-3140b7ce-6a6d-4692-a5f4-dc7a3b3f5441.png">

A quick count of the data points reveals that the data viz is only visualizing 7 data points, whereas the table contains 9 rows. When we are looking closer, we realize that the data labels of the "this" view represent the sum of the segment values of the stacked bar chart. This finding might trigger a re-count of the data point and we will end somewhere between 


+ 5, the data points of bar chart are used to "calculate" the tabular data points, plus two data points for the tick mark visualization
+ 6, 3 datapoints to visualize stack I and 1 data point to visualize stack II, where the data point is used to twice, plus two data points for the data visualization
+ 7,  7 independent data points are used
No matter the result of the counting the data points. It is obvious that the data inside the table has to be transformed. The next sections describe specific transformations to create the above visualization. As I will use this writing as a how-to on transformation for some future work it will also contain transformations that are not necessary to obtain the data visualization.

## General thoughts about data transformations
Data transformations are required to shape the available data in a way that a given body of data (basically the dataset) can be used to create a data visualization that helps us telling our data story conveys insights in an efficient way.
If you are wondering if you should use DAX to "shape" your data before you pass the data to the deneb visual or use vega/vega-lite transforms. My answer is simple: it depends. If you are sure that the DAX statement will be used more than once and you are not familiar with vega/vega-lite then of course use DAX. My experience dictates using the vega/vega-lite transformations though. I can tell they are fast and do not add measures to a data model that are only used in combination with specific data visualizations. Another reason why I do data transformations inside of deneb is reusability. I realized that it is often more simple to adapt a "complex" deneb specification to a given dataset, than to bring more complex DAX statements into various Power BI datasets.
A transform is specified either in the main specification, but it can also be specified inside an individual view if there is a multi-view composition, no matter if this composition is a multilayered view using "layer": ... or a multi-view composition, e.g., using "hconcat:" ... 
Some things to know about data transformations:

+ The transform array can contain multiple transformations. 
+ Transformations are executed in the order of definition. 
+ The result of a data transformation is available in subsequent data transformations.

One of the transformations that are described in this chapter is "rowIndexByGroupAndSegment". This column in combination with the filter transform helps to remove duplicate rows. It is very likely that real duplicate rows will not be present in your dataset when you are working with Power BI, the reason for this is that Power BI automatically removes duplicate rows by grouping across all rows, but without using aggregation functions for numeric values. Nevertheless, I tend to use various transforms at the view level and not in the encoding channel. Maybe this is only because of my data engineering experience, but I think that defining all transforms in a single place is reducing the overall complexity of the specification.

## The mental model of the data visualization (I will find a better place for this section in the not so distant future)
Dear reader, please excuse this section. I know that you are expecting data transformation related aspects, but as I'm revisiting this repo after quite some time I consider this important, and I'm pretty sure that I will put this section somewhere else when I do some re-factoring of this repo. 
Whenever I start something new using a technology or programming language that I do not speak fluently, I start with creating a mental model to discover obstacles early.
To create the above data visualization some challenges/obstacles are obvious:

+ The final data viz is constructed by aligning different visualization types, a table like structure on the left, a stacked bar chart with data labels in the middle and a table like structure on the right. All three visualizations have to use the data from the column "group" as a common y-axis
+ Data has to be aggregated to create the values for the tabular like visualization on the left
+ Data has to be aggregated to create the data labels for the three segments ("C", "D", "E")

If you are not familiar with the ![vega-lite text mark](https://vega.github.io/vega-lite/docs/text.html), then one task that has to be tackled that does not appear immediately before your third eye -  the placement of the data label inside each bar at the base (or left) of each segment. If you are using Power BI as I do, then we have to admit that we are a little spoiled in regards to the placement of the data labels. We simply turn on the property, then we tweak the placement a little inside the given boundaries, nevertheless, often I'm able to achieve what I want. If not, I use the one that comes closest to what I want. Now, using vega-lite it's time to face the hard reality, showing data labels is not a simple property, it requires thinking and some maths. The next section will describe all the necessary transformations (more or less) to determine the position for the text placement at the base of each segment.

As this section will move to a better place, the gist that creates the above data viz will stay, here is the link: ![multi-view composition - stacked-bullet chart](https://vega.github.io/editor/#/gist/30bb2932e8924bce69ec7d0fd050831e/multi view composition - stacked bullet chart.json)

## the text mark (this section will move to where it belongs, guess what - in the not so distant future)
The next image shows all transforms including the "filter" transform that is removing duplicate rows.  The column "xSegmentBase" might be considered the final transform, this column contains the values necessary to place the text (that's what a data label is) at the base (the left border of a horizontal bar) of each segment (column: a) inside a stack (column: group):

<img width="700" alt="transform - data object - multiview and stacked bullet-chart" src="https://user-images.githubusercontent.com/29025119/236117075-3e158042-5f43-4275-9f5b-3b7e97a8f297.png">

I think it's obvious that I start with the result, this is also a habit of mine as it helps me to stay focused. Here it measn that I describe the calculate transform first.

## the calculate transform ![calculate transform](https://vega.github.io/vega-lite/docs/calculate.html)
The position of a segment's base is determined by a simple subtraction: [the sum of all preceeding segments] - [the value of the current segment]. To create a new column and calculate a value for each row the transform "calculate" can be used, the next code snippet shows the transform:

```
{
  "calculate": "datum['runningSumBySegment'] - datum['segmentSum']",
  "as": "xSegmentBase"
}
```
The formula to calculate the wanted value is provided between quotation marks. The name of the new column is defined by using the key-word "as".
Referencing a column inside a data object is done by using datum. Datum can be used with many notations styles like datum.columnname or as I do datum['columnname']. To be honest, I'm not expert enough to recommend one over the other.

## the join aggregate ![join aggregate](https://vega.github.io/vega-lite/docs/joinaggregate.html)
JoinAggregate is a very powerful transform as it creates new values based on aggregations across multiple rows while preserving all rows. This transform is very useful if  detailValue / groupValue calculations are needed for the data visualization. Preserving rows of course, means repeating  values. This must not be fogotton. I will remove all unnecessary rows by using the filter transform, but this is a later step.
The next two code snippets show how the columns segmentSum and groupSum are calculated:

```
{
  "joinaggregate": [
    {
     "field": ["b"], 
     "op": ["sum"],
     "as": ["segmentSum"]
    }
  ],
  "groupby": ["group", "a"] 
}
```

```
{
  "joinaggregate": [
    {
      "field": ["b"], 
      "op": ["sum"], 
      "as": ["groupSum"]
    }
  ],
  "groupby": ["group"]
}
```

Looking at the syntax, it becomes obvious that each joinaggregate transformation is part of a list (the outer curly brackets). This list contains multiple transformations, where each transformation is an array. 
The next image shows how the transform works to create the column "segmentSum":

<img width="343" alt="transform - joinaggregate" src="https://user-images.githubusercontent.com/29025119/236118505-3e4fed28-dd7e-4d1d-a11d-32ad0c0f2592.png">

It's necessary to note that the order of the columns that are passed to the "groupby" phrase does not influence the result.

## the window transform ![window transform](https://vega.github.io/vega-lite/docs/window.html)
WINDOW adding new columns based on calculations for a sorted partition of a data object (a sorted group of rows). This is similar to the windowing functions you might know from SQL and can also be compared to the new DAX windowing functions. The next two code snippets show how the columns "rowIndexByGroupAndSegment" and "runningSumBySegment" are calculated:

```
{
  "window": [
    {
      "op": ["row_number"], 
      "as": ["rowIndexByGroupAndSegment"]
    }
  ], 
  "groupby": ["group", "a"]
}
```

```
{
  "window": [{
    "op": "sum",
    "field": "segmentSum",
    "as": "runningSumBySegment"
  }],
  "groupby": ["group"],
  "sort": [{"field": "rowIndexByGroup", "order": "ascending"}]
}
```

Due to the nature of the operator ("op", the operator determines the window function) the data field can be omitted, e.g. row_number does not require a data field.
The transform that creates the column "runningSumBySegment" is special in many cases, first it is demonstrating that columns can be referenced that have been created in preceeding transforms, but it is also using the power of the window transform. Here the data object is partitioned by the field "group", then the rows inside each partition are sorted by the field "rowIndexByGroup". The partitions are determined by the columns specified by the "groupby" operator. But the real "magic" of the window transform happens when the aggregation function (specified by the "op" operator) is applied to each row of the ordered partition. But this application is not only considering the current row, instead the aggregation function is applied to the current row and all preceeding rows, the preceedings rows are determined by the value of the column that is defined by the "sort" operator.  I always imagine the application of a window transform (but not only when I'm using vega or vega-lite) as an iterator. This iterator loops across all the rows inside a partition applies the aggregation function to a stack of values. Of course this stack is can be a single value, e.g. when the aggregation is more simple like count or sum, or a real stack of many values when the aggregation function is more complex like rank. I assume this is the reason why I consider the application of window transforms as some kind of magic, I do not have to implement the logic by myself I only use operators and magic is happening. The next image hopefully explains some of this magic:

<img width="425" alt="transform -window iterating across rows inside a partition" src="https://user-images.githubusercontent.com/29025119/236118887-007321a1-0041-40ad-886c-33bd5111fa46.png">

## the filter transform ![filter transform](https://vega.github.io/vega-lite/docs/filter.html)
The filter transform simply filters the data object, the next code snippet shows a very simple filter:

```
{
  "filter": "datum['rowIndexByGroupAndSegment']===1"
}
```

I very often use the transform operation "joinaggregate", this operation repeats an aggregated value for a given partition. This of course requires removing these duplicate values at a later stage. This is exactly what the above snippet is doing.

# The mark
# The encoding

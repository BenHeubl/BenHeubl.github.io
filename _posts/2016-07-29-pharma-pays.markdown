---
title: Pharma payments to doctors
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - Jekyll
tags:
  - update
---

Data by the Association of the British Pharmaceutical Industry reveals that: <http://www.abpi.org.uk/our-work/disclosure/Pages/DocumentLibrary.aspx?Paged=TRUE&p_SortBehavior=0&p_FileLeafRef=Sobi_MethodologicalNotes_2015%2epdf&p_ID=209&PageFirstRow=101&&View={67D056CF-A553-4724-A9E6-230AE9681875}#>

ft piece: <https://www.ft.com/content/b3e42806-3ec7-11e6-8716-a4a71e8140b0>

data analysis on UK payments. Questions:

- Which pharma companies stand out, and for what purpose.
- are there correlations
- draw conclusions to German's dataset

background: this is a list for individual recipients of non-research payments and benefits from drug companies.

I got a friend in Germany, a doctor and his stories about his annual medical trading courses make me depressed how good he has it. The food would not only be good but excellent, and the hotels are superb. Companies hosting these venues are spending a fortune on accommodating him and his medical colleagues. In recent years in the US, there have been increasing pressure on companies to publish their payments to doctors. Now thanks to Germany investigative news agency [Correctiv](https://correctiv.org/recherchen/euros-fuer-aerzte/artikel/2016/07/26/keiner-ist-so-nett-wie-der-pharmareferent/), the trend towards more openness comes to Germany.

Of course nothing in life comes for free, and pharmaceutical companies do much to recommend their products to doctor. A new dataset appeared on the horizon including micro data for 2015 payments from the top pharmaceutical companies to Germany's healthcare professionals and organisations. Collected and cleaned by Correctiv, the data roughly represents 30 percent of all payments, according to one of the members of Correctiv.

Looking at the largest payments, Baxter-Deutschland paid 1,166,666 Euros under the label "donations/grants" to a organisation called German association fighting malnutrition which is based in Bonn, last year. This is indeed an outlier.

![pic1]({{ site.url }}/images/pharma/plots/A.svg)

When looking at all payments, we can clearly see that some companies do pay aggressively more than others. Companies such as Boehringer Ingelheim, Astrazeneca and Novartis seem to have by far the widest spread.

![pic1]({{ site.url }}/images/pharma/plots/B.svg)

The average sum for donation payments are far higher that other payments types. While this is not surprising, the outliers in covering individual fees is.

![pic1]({{ site.url }}/images/pharma/plots/C.svg)

Also interesting is the maximum payments of companies. However, this might not be the best clue to judge companies of wrongdoing, as ever small amounts can already influence healthcare professionals. News companies such as Propublica could went into detail on how much payments could influence what drugs doctors advice to their patients (read more [here](https://correctiv.org/recherchen/euros-fuer-aerzte/artikel/2016/07/26/keiner-ist-so-nett-wie-der-pharmareferent/)).

![pic1]({{ site.url }}/images/pharma/plots/D.svg)

The winner in paying the highest average amount is pharma company Baxalta. With more than 6.647 Euro in average payments (more than two third higher than the leading second and third company), the company stood out. It also apparently claims the title to be one of the leading biopharmaceutical company advancing innovative therapies in haematology, immunology, and oncology. How much its aggressive payment strategy has helped to its success is left to be evaluated.

![pic1]({{ site.url }}/images/pharma/plots/E.svg)

# Now to the predictive modelling stuff

Since only roughly 30 percent of the micro data is out, wouldn't it be interesting to build a model to be able to predict datapoint, if we could get our hands on model data. One question that bugged me is what are the chances that...

subheaderintro:

intro:

main:

wrapup/concl:

call to action:

<!-- <link rel="stylesheet" type="text/css" href="/javascripts/posts/test/style.css"> <script src="/javascripts/libs/d3.4.11.js" type="text/javascript"></script> <script src="/javascripts/libs/lodash.js" type="text/javascript"></script> <script src="/javascripts/libs/d3-jetpack-v1.js" type="text/javascript"></script> <script src="/javascripts/libs/d3-starterkit-v0.js" type="text/javascript"></script> <script src="/javascripts/posts/test/graphtest.js"></script> -->

<link href="/bootstrap.min.css" rel="stylesheet">



<link href="/clean-blog.css" rel="stylesheet">



<link href="font-awesome.min.css" rel="stylesheet" type="text/css">



<link href="http://fonts.googleapis.com/css?family=Lora:400,700,400italic,700italic" rel="stylesheet" type="text/css">



<link href="http://fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,600italic,700italic,800italic,400,300,600,700,800" rel="stylesheet" type="text/css">



<script src="/jquery.min.js">
</script>



<script src="/bootstrap.min.js">
</script>



<script src="/jqBootstrapValidation.js">
</script>



<script src="/contact_me.js">
</script>



<script src="/clean-blog.min.js">
</script>

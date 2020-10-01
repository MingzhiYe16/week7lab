Lab 07 - Web scraping and Regular Expressions
================

``` r
knitr::opts_chunk$set(include  = TRUE)
```

Learning goals
==============

-   Use a real world API to make queries and process the data.
-   Use regular expressions to parse the information.
-   Practice your GitHub skills.

Lab description
===============

In this lab, we will be working with the [NCBI API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and extract information using XML and regular expressions. For this lab, we will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document` document.

Question 1: How many sars-cov-2 papers?
---------------------------------------

Build an automatic counter of sars-cov-2 papers using PubMed. You will need to apply XPath as we did during the lecture to extract the number of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/span
")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "33,814"

Don't forget to commit your work!

Question 2: Academic publications on COVID19 and Hawaii
-------------------------------------------------------

You need to query the following The parameters passed to the query are documented [here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

Use the function `httr::GET()` to make the following query:

1.  Baseline URL: <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:

    -   db: pubmed
    -   term: covid19 hawaii
    -   retmax: 1000

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db= "pubmed",
    term= "covid19 hawaii",
    retmax= 1000)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character list to analyze the text directly with `as.character()`. Another way of processing the data could be using lists with the function `xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don't forget to commit and push your results to your GitHub repo!).

Question 3: Get details about the articles
------------------------------------------

The Ids are wrapped around text in the following way: `<Id>... id number ...</Id>`. we can use a regular expression that extract that information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids [[1]] can change list to character
ids <- stringr::str_extract_all(ids, "<Id>[1-9]+</Id>")[[1]]



# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>")
ids <- stringr::str_remove_all(ids, "</Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers. As before, we will need to coerce the contents (results) to a list using:

1.  Baseline url: <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:
    -   db: pubmed
    -   id: A character with all the ids separated by comma, e.g., "1232131,546464,13131"
    -   retmax: 1000
    -   rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it around `I()` (as you would do in a formula in R). For example, the text `"123,456"` is replaced with `"123%2C456"`. If you don't want that behavior, you would need to do the following `I("123,456")`.

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = paste(ids, collapse = ","),
    retmax = 1000,
    rettype = "abstract"
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time for committing and pushing your work!

Question 4: Distribution of universities, schools, and departments
------------------------------------------------------------------

Using the function `stringr::str_extract_all()` applied on `publications_txt`, capture all the terms of the form:

1.  University of ...
2.  ... Institute of ...

Write a regular expression that captures all such instances

``` r
library(stringr)
institution <- str_extract_all(
  publications_txt,
  "University\\s+of\\s+[[:alpha:]]+|[[:alpha:]]+\\s+Institute\\s+of\\s+[[:alpha:]]+"
  ) 

institution <- unlist(institution)
table(institution)
```

    ## institution
    ## National Institute of Environmental               University of Arizona 
    ##                                   3                                   2 
    ##            University of California              University of Colorado 
    ##                                   4                                   1 
    ##                 University of Hawai                University of Hawaii 
    ##                                  13                                  22 
    ##                University of Health                  University of Iowa 
    ##                                   1                                   1 
    ##                   University of New          University of Pennsylvania 
    ##                                   2                                  18 
    ##            University of Pittsburgh               University of Science 
    ##                                   1                                  14 
    ##                 University of South              University of Southern 
    ##                                   1                                   1 
    ##                 University of Texas                   University of the 
    ##                                   2                                   1 
    ##                  University of Utah 
    ##                                   1

Repeat the exercise and this time focus on schools and departments in the form of

1.  School of ...
2.  Department of ...

And tabulate the results

``` r
schools_and_deps <- str_extract_all(
  publications_txt,
  "School of\\s[[:alpha:]]+|Department of\\s[[:alpha:]]+"
  )
schools_and_deps<-unlist(schools_and_deps)
table(schools_and_deps)
```

    ## schools_and_deps
    ## Department of Anesthesiology     Department of Cardiology 
    ##                            3                            1 
    ##           Department of Cell  Department of Communication 
    ##                            4                            1 
    ##  Department of Computational  Department of Environmental 
    ##                            1                            1 
    ##   Department of Epidemiology         Department of Family 
    ##                            9                            3 
    ##        Department of Genetic      Department of Geography 
    ##                            1                            2 
    ##     Department of Infectious       Department of Internal 
    ##                            2                            1 
    ##        Department of Medical       Department of Medicine 
    ##                            3                           43 
    ##         Department of Native     Department of Nephrology 
    ##                            2                            5 
    ##      Department of Nutrition Department of Otolaryngology 
    ##                            4                            4 
    ##     Department of Pediatrics       Department of Physical 
    ##                            9                            3 
    ##     Department of Preventive     Department of Psychiatry 
    ##                            2                            4 
    ##     Department of Psychology   Department of Quantitative 
    ##                            1                            5 
    ## Department of Rehabilitation         Department of Social 
    ##                            1                            1 
    ##        Department of Surgery       Department of Tropical 
    ##                            2                            4 
    ##           Department of Twin        Department of Urology 
    ##                            2                            1 
    ##       Department of Veterans           School of Medicine 
    ##                            2                           66 
    ##            School of Natural             School of Public 
    ##                            1                           14

Question 5: Form a database
---------------------------

We want to build a dataset which includes the title and the abstract of the paper. The title of all records is enclosed by the HTML tag `ArticleTitle`, and the abstract by `Abstract`.

Before applying the functions to extract text directly, it will help to process the XML a bit. We will use the `xml2::xml_children()` function to keep one element per id. This way, if a paper is missing the abstract, or something else, we will be able to properly match PUBMED IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements of `pub_char_list`. You can either use `sapply()` as we just did, or simply take advantage of vectorization of `stringr::str_extract`

``` r
abstracts <- str_extract(pub_char_list, "<Abstract>(\\n|.)+</Abstract>")
abstracts <- str_remove_all(abstracts, "</?[[:alnum:]]+>")
abstracts <- str_replace_all(abstracts, "\\s+"," ")
table(is.na(abstracts))
```

    ## 
    ## FALSE  TRUE 
    ##    13     9

How many of these don't have an abstract? Now, the title

``` r
titles <- str_extract(pub_char_list, "<ArticleTitle>(\\n|.)+</ArticleTitle>")
titles <- str_remove_all(titles, "</?[[:alnum:]]+>")
titles <- str_replace_all(titles, "\\s+"," ")
```

Finally, put everything together into a single `data.frame` and use `knitr::kable` to print the results

``` r
database <- data.frame(
  PubMedID = ids,
  Title = titles,
  Abstracts=abstracts
)
knitr::kable(database)
```

<table>
<colgroup>
<col width="0%" />
<col width="6%" />
<col width="93%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">PubMedID</th>
<th align="left">Title</th>
<th align="left">Abstracts</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">32921878</td>
<td align="left">Will COVID-19 be one shock too many for smallholder coffee livelihoods?</td>
<td align="left">Coffee supports the livelihoods of millions of smallholder farmers in more than 52 countries, and generates billions of dollars in revenue. The threats that COVID-19 pose to the global coffee sector is daunting with profound implications for coffee production. The financial impacts will be long-lived and uneven, and smallholders will be among the hardest hit. We argue that the impacts are rooted in the systemic vulnerability of the coffee production system and the unequal ways the sector is organized: Large revenues from the sale of coffee in the Global North are made possible by mostly impoverished smallholders in the Global South. COVID-19 will accentuate the existing vulnerabilities and create new ones, forcing many smallholders into alternative livelihoods. This outcome, however, is not inevitable. COVID-19 presents an opportunity to rebalance the system that currently creates large profits on one end of the supply chain and great vulnerability on the other. <U+00A9> 2020 Elsevier Ltd. All rights reserved.</td>
</tr>
<tr class="even">
<td align="left">32912595</td>
<td align="left">Ensuring mental health access for vulnerable populations in COVID era.</td>
<td align="left">NA</td>
</tr>
<tr class="odd">
<td align="left">32881116</td>
<td align="left">Delivering Prolonged Exposure Therapy via Videoconferencing During the COVID-19 Pandemic: An Overview of the Research and Special Considerations for Providers.</td>
<td align="left">Leveraging technology to provide evidence-based therapy for posttraumatic stress disorder (PTSD), such as prolonged exposure (PE), during the COVID-19 pandemic helps ensure continued access to first-line PTSD treatment. Clinical video teleconferencing (CVT) technology can be used to effectively deliver PE while reducing the risk of COVID-19 exposure during the pandemic for both providers and patients. However, provider knowledge, experience, and comfort level with delivering mental health care services, such as PE, via CVT is critical to ensure a smooth, safe, and effective transition to virtual care. Further, some of the limitations associated with the pandemic, including stay-at-home orders and physical distancing, require that providers become adept at applying principles of exposure therapy with more flexibility and creativity, such as when assigning in vivo exposures. The present paper provides the rationale and guidelines for implementing PE via CVT during COVID-19 and includes practical suggestions and clinical recommendations. Published 2020. This article is a U.S. Government work and is in the public domain in the USA.</td>
</tr>
<tr class="even">
<td align="left">32763956</td>
<td align="left">Reactive arthritis after COVID-19 infection.</td>
<td align="left">Reactive arthritis (ReA) is typically preceded by sexually transmitted disease or gastrointestinal infection. An association has also been reported with bacterial and viral respiratory infections. Herein, we report the first case of ReA after the he severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) infection. This male patient is in his 50s who was admitted with COVID-19 pneumonia. On the second day of admission, SARS-CoV-2 PCR was positive from nasopharyngeal swab specimen. Despite starting standard dose of favipiravir, his respiratory condition deteriorated during hospitalisation. On the fourth hospital day, he developed acute respiratory distress syndrome and was intubated. On day 11, he was successfully extubated, subsequently completing a 14-day course of favipiravir. On day 21, 1 day after starting physical therapy, he developed acute bilateral arthritis in his ankles, with mild enthesitis in his right Achilles tendon, without rash, conjunctivitis, or preceding diarrhoea or urethritis. Arthrocentesis of his left ankle revealed mild inflammatory fluid without monosodium urate or calcium pyrophosphate crystals. Culture of synovial fluid was negative. Plain X-rays of his ankles and feet showed no erosive changes or enthesophytes. Tests for syphilis, HIV, anti-streptolysin O (ASO), Mycoplasma, Chlamydia pneumoniae, antinuclear antibody, rheumatoid factor, anticyclic citrullinated peptide antibody and Human Leukocyte Antigen-B27 (HLA-B27) were negative. Gonococcal and Chlamydia trachomatis urine PCR were also negative. He was diagnosed with ReA. Nonsteroidal Anti-Inflammatory Drug (NSAID)s and intra-articular corticosteroid injection resulted in moderate improvement. <U+00A9> Author(s) (or their employer(s)) 2020. Re-use permitted under CC BY-NC. No commercial re-use. See rights and permissions. Published by BMJ.</td>
</tr>
<tr class="odd">
<td align="left">32742897</td>
<td align="left">COVID-19 outbreak on the Diamond Princess Cruise Ship in February 2020.</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">32649272</td>
<td align="left">Combating COVID-19 and Building Immune Resilience: A Potential Role for Magnesium Nutrition?</td>
<td align="left"><AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">In December 2019, the viral pandemic of respiratory illness caused by COVID-19 began sweeping its way across the globe. Several aspects of this infectious disease mimic metabolic events shown to occur during latent subclinical magnesium deficiency. Hypomagnesemia is a relatively common clinical occurrence that often goes unrecognized since magnesium levels are rarely monitored in the clinical setting. Magnesium is the second most abundant intracellular cation after potassium. It is involved in &gt;600 enzymatic reactions in the body, including those contributing to the exaggerated immune and inflammatory responses exhibited by COVID-19 patients. <AbstractText Label="METHODS" NlmCategory="METHODS">A summary of experimental findings and knowledge of the biochemical role magnesium may play in the pathogenesis of COVID-19 is presented in this perspective. The National Academy of Medicine's Standards for Systematic Reviews were independently employed to identify clinical and prospective cohort studies assessing the relationship of magnesium with interleukin-6, a prominent drug target for treating COVID-19. <AbstractText Label="RESULTS" NlmCategory="RESULTS">Clinical recommendations are given for prevention and treatment of COVID-19. Constant monitoring of ionized magnesium status with subsequent repletion, when appropriate, may be an effective strategy to influence disease contraction and progression. The peer-reviewed literature supports that several aspects of magnesium nutrition warrant clinical consideration. Mechanisms include its &quot;calcium-channel blocking&quot; effects that lead to downstream suppression of nuclear factor-Kβ, interleukin-6, c-reactive protein, and other related endocrine disrupters; its role in regulating renal potassium loss; and its ability to activate and enhance the functionality of vitamin D, among others. <AbstractText Label="CONCLUSION" NlmCategory="CONCLUSIONS">As the world awaits an effective vaccine, nutrition plays an important and safe role in helping mitigate patient morbidity and mortality. Our group is working with the Academy of Nutrition and Dietetics to collect patient-level data from intensive care units across the United States to better understand nutrition care practices that lead to better outcomes.</td>
</tr>
<tr class="odd">
<td align="left">32596689</td>
<td align="left">Viewpoint: Pacific Voyages - Ships - Pacific Communities: A Framework for COVID-19 Prevention and Control.</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">32592394</td>
<td align="left">A role for selenium-dependent GPX1 in SARS-CoV-2 virulence.</td>
<td align="left">NA</td>
</tr>
<tr class="odd">
<td align="left">32584245</td>
<td align="left">Communicating Effectively With Hospitalized Patients and Families During the COVID-19 Pandemic.</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">32486844</td>
<td align="left">Covid-19 and Diabetes in Hawaii.</td>
<td align="left">NA</td>
</tr>
<tr class="odd">
<td align="left">32462545</td>
<td align="left">Treatments Administered to the First 9152 Reported Cases of COVID-19: A Systematic Review.</td>
<td align="left">The emergence of SARS-CoV-2/2019 novel coronavirus (COVID-19) has created a global pandemic with no approved treatments or vaccines. Many treatments have already been administered to COVID-19 patients but have not been systematically evaluated. We performed a systematic literature review to identify all treatments reported to be administered to COVID-19 patients and to assess time to clinically meaningful response for treatments with sufficient data. We searched PubMed, BioRxiv, MedRxiv, and ChinaXiv for articles reporting treatments for COVID-19 patients published between 1 December 2019 and 27 March 2020. Data were analyzed descriptively. Of the 2706 articles identified, 155 studies met the inclusion criteria, comprising 9152 patients. The cohort was 45.4% female and 98.3% hospitalized, and mean (SD) age was 44.4 years (SD 21.0). The most frequently administered drug classes were antivirals, antibiotics, and corticosteroids, and of the 115 reported drugs, the most frequently administered was combination lopinavir/ritonavir, which was associated with a time to clinically meaningful response (complete symptom resolution or hospital discharge) of 11.7 (1.09) days. There were insufficient data to compare across treatments. Many treatments have been administered to the first 9152 reported cases of COVID-19. These data serve as the basis for an open-source registry of all reported treatments given to COVID-19 patients at www.CDCN.org/CORONA . Further work is needed to prioritize drugs for investigation in well-controlled clinical trials and treatment protocols.</td>
</tr>
<tr class="even">
<td align="left">32432219</td>
<td align="left">Insights in Public Health: COVID-19 Special Column: The Crisis of Non-Communicable Diseases in the Pacific and the Coronavirus Disease 2019 Pandemic.</td>
<td align="left">Globally, coronavirus disease 2019 (COVID-19) is threatening human health and changing the way people live. With the increasing evidence showing comorbidities of COVID-19 and non-communicable diseases (NCDs), the Pacific region, where approximately 75% of deaths are due to NCDs, is significantly vulnerable during this crisis unless urgent action is taken. Whilst enforcing the critical mitigation measures of the COVID-19 pandemic in the Pacific, it is also paramount to incorporate and strengthen NCD prevention and control measures to safeguard people with NCDs and the general population; keep people healthy and minimise the impact of COVID-19. To sustain wellbeing of health, social relationships, and the economy in the Pacific, it is a critical time for all governments, development partners and civil societies to show regional solidarity in the fight against emerging COVID-19 health crisis and existing Pacific NCDs crisis through a whole of government and whole of society approach. <U+00A9>Copyright 2020 by University Health Partners of Hawai‘i (UHP Hawai‘i).</td>
</tr>
<tr class="odd">
<td align="left">32432218</td>
<td align="left">COVID-19 Special Column: COVID-19 Hits Native Hawaiian and Pacific Islander Communities the Hardest.</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">32432217</td>
<td align="left">COVID-19 Special Column: Principles Behind the Technology for Detecting SARS-CoV-2, the Cause of COVID-19.</td>
<td align="left">Nationwide shortages of tests that detect severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) and diagnose coronavirus disease 2019 (COVID-19) have led the US Food and Drug Administration (FDA) to significantly relax regulations regarding COVID-19 diagnostic testing. To date the FDA has given emergency use authorization (EUA) to 48 COVID-19 in vitro diagnostic tests and 21 high complexity molecular-based laboratory developed tests, as well as implemented policies that give broad authority to clinical laboratories and commercial manufacturers in the development, distribution, and use of COVID-19 diagnostic tests. Currently, there are 2 types of diagnostic tests available for the detection of SARS-CoV-2: (1) molecular and (2) serological tests. Molecular detection of nucleic acid (RNA or DNA) sequences relating to the suspected pathogen is indicative of an active infection with the suspected pathogen. Serological tests detect antibodies against the suspected pathogen, which are produced by an individual's immune system. A positive serological test result indicates recent exposure to the suspected pathogen but cannot be used to determine if the individual is actively infected with the pathogen or immune to reinfection. In this article, the SARS-CoV-2 diagnostic tests currently approved by the FDA under EUA are reviewed, and other diagnostic tests that researchers are developing to detect SARS-CoV-2 infection are discussed. <U+00A9>Copyright 2020 by University Health Partners of Hawai‘i (UHP Hawai‘i).</td>
</tr>
<tr class="odd">
<td align="left">32427288</td>
<td align="left">ACE2 receptor expression in testes: implications in coronavirus disease 2019 pathogenesis<U+2020>.</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">32386898</td>
<td align="left">Geospatial analysis of COVID-19 and otolaryngologists above age 60.</td>
<td align="left"><AbstractText Label="OBJECTIVE" NlmCategory="OBJECTIVE">The 2019 novel coronavirus (COVID-19) is disproportionately impacting older individuals and healthcare workers. Otolaryngologists are especially susceptible with the elevated risk of aerosolization and corresponding high viral loads. This study utilizes a geospatial analysis to illustrate the comparative risks of older otolaryngologists across the United States during the COVID-19 pandemic. <AbstractText Label="METHODS AND MATERIALS" NlmCategory="METHODS">Demographic and state population data were extracted from the State Physician Workforce Reports published by the AAMC for the year 2018. A geospatial heat map of the United States was then constructed to illustrate the location of COVID-19 confirmed case counts and the distributions of ENTs over 60 years for each state. <AbstractText Label="RESULTS" NlmCategory="RESULTS">In 2018, out of a total of 9578 practicing U.S. ENT surgeons, 3081 were older than 60 years (32.2%). The states with the highest proportion of ENTs over 60 were Maine, Delaware, Hawaii, and Louisiana. The states with the highest ratios of confirmed COVID-19 cases to the number of total ENTs over 60 were New York, New Jersey, Massachusetts, and Michigan. <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">Based on our models, New York, New Jersey, Massachusetts, and Michigan represent states where older ENTs may be the most susceptible to developing severe complications from nosocomial transmission of COVID-19 due to a combination of high COVID-19 case volumes and a high proportion of ENTs over 60 years. Copyright <U+00A9> 2020 Elsevier Inc. All rights reserved.</td>
</tr>
<tr class="odd">
<td align="left">32371624</td>
<td align="left">The War on COVID-19 Pandemic: Role of Rehabilitation Professionals and Hospitals.</td>
<td align="left">The global outbreak of coronavirus disease 2019 has created an unprecedented challenge to the society. Currently, the United States stands as the most affected country, and the entire healthcare system is affected, from emergency department, intensive care unit, postacute care, outpatient, to home care. Considering the debility, neurological, pulmonary, neuromuscular, and cognitive complications, rehabilitation professionals can play an important role in the recovery process for individuals with coronavirus disease 2019. Clinicians across the nation's rehabilitation system have already begun working to initiate intensive care unit-based rehabilitation care and develop programs, settings, and specialized care to meet the short- and long-term needs of these individuals. We describe the anticipated rehabilitation demands and the strategies to meet the needs of this population. The complications from coronavirus disease 2019 can be reduced by (1) delivering interdisciplinary rehabilitation that is initiated early and continued throughout the acute hospital stay, (2) providing patient/family education for self-care after discharge from inpatient rehabilitation at either acute or subacute settings, and (3) continuing rehabilitation care in the outpatient setting and at home through ongoing therapy either in-person or via telehealth.</td>
</tr>
<tr class="even">
<td align="left">32371551</td>
<td align="left">The COronavirus Pandemic Epidemiology (COPE) Consortium: A Call to Action.</td>
<td align="left">The rapid pace of the severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2; COVID-19) pandemic presents challenges to the real-time collection of population-scale data to inform near-term public health needs as well as future investigations. We established the COronavirus Pandemic Epidemiology (COPE) consortium to address this unprecedented crisis on behalf of the epidemiology research community. As a central component of this initiative, we have developed a COVID Symptom Study (previously known as the COVID Symptom Tracker) mobile application as a common data collection tool for epidemiologic cohort studies with active study participants. This mobile application collects information on risk factors, daily symptoms, and outcomes through a user-friendly interface that minimizes participant burden. Combined with our efforts within the general population, data collected from nearly 3 million participants in the United States and United Kingdom are being used to address critical needs in the emergency response, including identifying potential hot spots of disease and clinically actionable risk factors. The linkage of symptom data collected in the app with information and biospecimens already collected in epidemiology cohorts will position us to address key questions related to diet, lifestyle, environmental, and socioeconomic factors on susceptibility to COVID-19, clinical outcomes related to infection, and long-term physical, mental health, and financial sequalae. We call upon additional epidemiology cohorts to join this collective effort to strengthen our impact on the current health crisis and generate a new model for a collaborative and nimble research infrastructure that will lead to more rapid translation of our work for the betterment of public health. <U+00A9>2020 American Association for Cancer Research.</td>
</tr>
<tr class="odd">
<td align="left">32361738</td>
<td align="left">Risk Factors Associated with Clinical Outcomes in 323 COVID-19 Hospitalized Patients in Wuhan, China.</td>
<td align="left"><AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">With evidence of sustained transmission in more than 190 countries, coronavirus disease 2019 (COVID-19) has been declared a global pandemic. Data are urgently needed about risk factors associated with clinical outcomes. <AbstractText Label="METHODS" NlmCategory="METHODS">A retrospective review of 323 hospitalized patients with COVID-19 in Wuhan was conducted. Patients were classified into three disease severity groups (non-severe, severe, and critical), based on initial clinical presentation. Clinical outcomes were designated as favorable and unfavorable, based on disease progression and response to treatments. Logistic regression models were performed to identify risk factors associated with clinical outcomes, and log-rank test was conducted for the association with clinical progression. <AbstractText Label="RESULTS" NlmCategory="RESULTS">Current standard treatments did not show significant improvement in patient outcomes. By univariate logistic regression analysis, 27 risk factors were significantly associated with clinical outcomes. Multivariate regression indicated age over 65 years (p&lt;0.001), smoking (p=0.001), critical disease status (p=0.002), diabetes (p=0.025), high hypersensitive troponin I (&gt;0.04 pg/mL, p=0.02), leukocytosis (&gt;10 x 109/L, p&lt;0.001) and neutrophilia (&gt;75 x 109/L, p&lt;0.001) predicted unfavorable clinical outcomes. By contrast, the administration of hypnotics was significantly associated with favorable outcomes (p&lt;0.001), which was confirmed by survival analysis. <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">Hypnotics may be an effective ancillary treatment for COVID-19. We also found novel risk factors, such as higher hypersensitive troponin I, predicted poor clinical outcomes. Overall, our study provides useful data to guide early clinical decision making to reduce mortality and improve clinical outcomes of COVID-19. <U+00A9> The Author(s) 2020. Published by Oxford University Press for the Infectious Diseases Society of America. All rights reserved. For permissions, e-mail: <a href="mailto:journals.permissions@oup.com">journals.permissions@oup.com</a>.</td>
</tr>
<tr class="even">
<td align="left">32326959</td>
<td align="left">High-flow nasal cannula may be no safer than non-invasive positive pressure ventilation for COVID-19 patients.</td>
<td align="left">NA</td>
</tr>
<tr class="odd">
<td align="left">32314954</td>
<td align="left">COVID-19 Community Stabilization and Sustainability Framework: An Integration of the Maslow Hierarchy of Needs and Social Determinants of Health.</td>
<td align="left">All levels of government are authorized to apply coronavirus disease 2019 (COVID-19) protection measures; however, they must consider how and when to ease lockdown restrictions to limit long-term societal harm and societal instability. Leaders that use a well-considered framework with an incremental approach will be able to gradually restart society while simultaneously maintaining the public health benefits achieved through lockdown measures. Economically vulnerable populations cannot endure long-term lockdown, and most countries lack the ability to maintain a full nationwide relief operation. Decision-makers need to understand this risk and how the Maslow hierarchy of needs and the social determinants of health can guide whole of society policies. Aligning decisions with societal needs will help ensure all segments of society are catered to and met while managing the crisis. This must inform the process of incremental easing of lockdowns to facilitate the resumption of community foundations, such as commerce, education, and employment in a manner that protects those most vulnerable to COVID-19. This study proposes a framework for identifying a path forward. It reflects on baseline requirements, regulations and recommendations, triggers, and implementation. Those desiring a successful recovery from the COVID-19 pandemic need to adopt an evidence-based framework now to ensure community stabilization and sustainability.</td>
</tr>
<tr class="even">
<td align="left">32259247</td>
<td align="left">Pain Management Best Practices from Multispecialty Organizations During the COVID-19 Pandemic and Public Health Crises.</td>
<td align="left"><AbstractText Label="BACKGROUND">It is nearly impossible to overestimate the burden of chronic pain, which is associated with enormous personal and socioeconomic costs. Chronic pain is the leading cause of disability in the world, is associated with multiple psychiatric comorbidities, and has been causally linked to the opioid crisis. Access to pain treatment has been called a fundamental human right by numerous organizations. The current COVID-19 pandemic has strained medical resources, creating a dilemma for physicians charged with the responsibility to limit spread of the contagion and to treat the patients they are entrusted to care for. <AbstractText Label="METHODS">To address these issues, an expert panel was convened that included pain management experts from the military, Veterans Health Administration, and academia. Endorsement from stakeholder societies was sought upon completion of the document within a one-week period. <AbstractText Label="RESULTS">In these guidelines, we provide a framework for pain practitioners and institutions to balance the often-conflicting goals of risk mitigation for health care providers, risk mitigation for patients, conservation of resources, and access to pain management services. Specific issues discussed include general and intervention-specific risk mitigation, patient flow issues and staffing plans, telemedicine options, triaging recommendations, strategies to reduce psychological sequelae in health care providers, and resource utilization. <AbstractText Label="CONCLUSIONS">The COVID-19 public health crisis has strained health care systems, creating a conundrum for patients, pain medicine practitioners, hospital leaders, and regulatory officials. Although this document provides a framework for pain management services, systems-wide and individual decisions must take into account clinical considerations, regional health conditions, government and hospital directives, resource availability, and the welfare of health care providers. The Author(s) 2020. Published by Oxford University Press on behalf of the American Academy of Pain Medicine. This work is written by a US Government employee and is in the public domain in the US.</td>
</tr>
</tbody>
</table>

Done! Knit the document, commit, and push.

Final Pro Tip (optional)
------------------------

You can still share the HTML document on github. You can include a link in your `README.md` file as the following:

``` md
View [here](https://ghcdn.rawgit.org/:user/:repo/:tag/:file)
```

For example, if we wanted to add a direct link the HTML page of lecture 7, we could do something like the following:

``` md
View [here](https://ghcdn.rawgit.org/USCbiostats/PM566/master/static/slides/07-apis-regex/slides.html)
```

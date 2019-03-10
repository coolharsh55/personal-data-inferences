# Personal Data Inference

Inferring additional personal data based on available data based on documented
methods. These methods might have additional attributes and dimensions such as
granularity and frequency. The project aims to find applicable inferences using
a semantic-web approach (e.g. OWL2 reasoner).

Summary of derivations in literature: [derivations.pdf](./derivations.pdf)

## How-To

### Creating Inferences

1. Create a subclass of `InferenceVector` from inferences ontology
2. Declare the inference using `owl:EquivalenClass` 
   1. Add annotation using `:infersData` to specify what personal data it infers
   2. Add other properties such as `rdfs:label` and `rdfs:isDefinedBy`
   3. below example for an inference based on data sourced from Twitter or Facebook

```OWL
:InferenceVector and (:usesData some(
	(pinfer-dimen:hasSource some pinfer-dimen:Twitter) or
	(pinfer-dimen:hasSource some pinfer-dimen:Facebook)
	))
```

3. Protege doesn't allow punning, so this is the less-elegant solution to get it working in Protege
   1. Declare instances of data items to use
   2. For the above, declare an instance of _Twitter_ OR _Facebook_ (choose one to show it works with either)
4. Declare data collected as `data-item` or anything
   1. Add `pinfer-dimen:hasSource` _Twitter_ or _Facebook_
   2. Add any other properties such as quality, type etc.
5. Create an instance of type `InstanceVector` e.g. `inference1`
   1. add property `usesData` with object defined above as `data-item`
   2. add other data items as needed
6. Run reasoner
7. The types inferred over `inference1` will be all the applicable inferences for the selected data.

Example from file:

```OWL
# infer privacy preferences from text and profile of Social Media Websites
Class: InferPrivacyPrefFromTwitter
    Annotations: 
        infersData <https://w3id.org/pinfer/personal-data#PrivacyPreferences>,
        rdfs:isDefinedBy <http://doi.acm.org/10.1145/3213586.3225228>
    EquivalentTo: 
        InferenceVector
         and (usesData some pinfer-pd:OnlineProfiles)
         and (usesData some 
            (pinfer-pd:Writings
             and (pinfer-dimen:hasSource some pinfer-dimen:Website)))
    SubClassOf: 
        InferenceVector

# create instance for inference that uses twitter posts and profiles
Individual: infer-privacy-pref-from-twitter
    Types: 
        InferenceVector
    Facts:  
     usesData  twitter-posts
     usesData  twitter-profiles

# twitter posts are a form of writing with source website Twitter 
Individual: twitter-posts
    Types: 
        pinfer-pd:Writings
    Facts:  
     pinfer-dimen:hasSource  pinfer-dimen:Twitter

# twitter profiles are a form of personal data
Individual: twitter-profiles
    Types: 
        pinfer-pd:TwitterData
        
# reasoner infers
infer-privacy-pref-from-twitter rdf:type InferPrivacyPrefFromTwitter
# therefore for the given data items, we can infer with the query -
SELECT ?data WHERE { infer-privacy-pref-from-twitter infersData ?data . }
# which results in -
<https://w3id.org/pinfer/personal-data#PrivacyPreferences>
```

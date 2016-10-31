# parsa
[ ![Codeship Status for amn41/parsa](https://app.codeship.com/projects/b06f6000-7444-0134-8053-76df66f7aa2d/status?branch=master)](https://app.codeship.com/projects/179147)

## Motivation

Parsa is a tool for intent classification and entity extraction. 
You can think of parsa as a set of high level APIs for building your own language parser using existing NLP and ML libraries.
The intended audience is mainly people developing bots. 
It can be used as a drop-in replacement for [wit](https://wit.ai) or [LUIS](https://luis.ai), but works as a local service rather than a web API. 

The setup process is designed to be as simple as possible. If you're currently using wit or LUIS, you just:
1. download your app data from wit or LUIS and feed it into parsa
2. run parsa on your machine and switch the URL of your wit/LUIS api calls to `localhost:5000/parse`.

Reasons you might use this over one of the aforementioned services: 
- you don't have to hand over your data to FB/MSFT/GOOG
- you don't have to make a `https` call every time.
- you can tune models to work well on your particular use case.

These points are laid out in more detail in a [blog post](https://medium.com/lastmile-conversations/do-it-yourself-nlp-for-bot-developers-2e2da2817f3d).

Parsa is written in Python, but it you can use it from any language through a HTTP API. 
If your project *is* written in Python you can simply import the relevant classes.
 
## Getting Started
```bash
python setup.py install
python -m parsa.server --mode=wit &
curl 'http://localhost:5000/parse?q=hello'
# returns e.g. '{"intent":"greet","entities":[]}'
```

There you go! you just parsed some text. Important command line options for `parsa.server` are as follows:
- mode: which service to emulate, can be 'wit' or 'luis', or just leave blank for default mode.
- path: dir where your trained models are saved. If you leave this blank parsa will just use a naive keyword matcher.



## Configuring a backend
Parsa itself doesn't have any external requirements, but in order to make it useful you need to install & configure a backend. 

#### MITIE

Currently, the only fully supported backend is [MITIE](https://github.com/mit-nlp/MITIE).

`pip install git+https://github.com/mit-nlp/MITIE.git`
and then download the [MITIE models](https://github.com/mit-nlp/MITIE/releases/download/v0.4/MITIE-models-v0.2.tar.bz2). The file you need is `total_word_feature_extractor.dat`

#### spaCy,  NLTK
Support for these NLP backends is in development and will be available soon:

- [spaCy](https://github.com/spacy-io/spaCy)
- [NLTK](www.nltk.org/)

NB that if you use spaCy or NLTK you will also need to use a separate machine learning library like scikit-learn or keras.

Install one of the above & then also a ML lib, e.g. scikit-learn or keras. 


## Creating your own language parser
### Cloning an existing wit or LUIS app:

Download your data from wit or LUIS. When you export your model from wit you will get a zipped directory. The file you need is `expressions.json`.
If you're exporting from LUIS you get a single json file, and that's the one you need. Create a config file (json format) like this one:

```json
{
  "path" : "/path/to/save/models/",
  "data" : "expressions.json",
  "backend" : "mitie",
  "backends" : {
    "mitie": {
      "fe_file":"/path/to/total_word_feature_extractor.dat"
    }
  }
}
```

and then pass this to the training script

```bash
python -m parsa.train -c config.json
```

you can also override any of the params in config.json with command line arguments.

### Running the server with your newly trained models

After training you will have a new dir containing your models, e.g. `/path/to/save/models/model_XXXXXX`. 
Just pass this path to the `parsa.server` script:

```bash
python -m parsa.server --mode=wit -p '/path/to/save/models/model_XXXXXX'
```


### Using Parsa from python
Pretty simple really, just open your python interpreter and type:
```python
from parsa.backends import MITIEInterpreter
interpreter = MITIEInterpreter('data/intent_classifier.dat','data/ner.dat','data/total_word_feature_extractor.dat')
interpreter.parse("hello world")  # -> {'intent':'greet','entities':[]}
```


## Roadmap 
- full support for spaCy backend
- entity normalisation: as is, the named entity extractor will happily extract `cheap` & `inexpensive` as entities of the `expense` class, but will not tell you that these are realisations of the same underlying concept. You can easily handle that with a list of aliases in your code, but we want to offer a more elegant & generalisable solution.
- parsing structured data, e.g. dates. We might use [parsedatetime](https://pypi.python.org/pypi/parsedatetime/) or possibly wit.ai's very own [duckling](https://duckling.wit.ai/). 
- support for more languages

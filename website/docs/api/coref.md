---
title: CoreferenceResolver
tag: class,experimental
source: spacy-experimental/coref/coref_component.py
teaser: 'Pipeline component for word-level coreference resolution'
api_base_class: /api/pipe
api_string_name: coref
api_trainable: true
---

> #### Installation
>
> ```bash
> $ pip install -U spacy-experimental
> ```

<Infobox title="Important note" variant="warning">

This component is not yet integrated into spaCy core, and is available via the
extension package
[`spacy-experimental`](https://github.com/explosion/spacy-experimental) starting
in version 0.6.0. It exposes the component via
[entry points](/usage/saving-loading/#entry-points), so if you have the package
installed, using `factory = "experimental_coref"` in your
[training config](/usage/training#config) or
`nlp.add_pipe("experimental_coref")` will work out-of-the-box.

</Infobox>

A `CoreferenceResolver` component groups tokens into clusters that refer to the
same thing. Clusters are represented as SpanGroups that start with a prefix
(`coref_clusters` by default).

A `CoreferenceResolver` component can be paired with a
[`SpanResolver`](/api/span-resolver) to expand single tokens to spans.

## Assigned Attributes {#assigned-attributes}

Predictions will be saved to `Doc.spans` as a [`SpanGroup`](/api/spangroup). The
span key will be a prefix plus a serial number referring to the coreference
cluster, starting from zero.

The span key prefix defaults to `"coref_clusters"`, but can be passed as a
parameter.

| Location                                   | Value                                                                                                   |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| `Doc.spans[prefix + "_" + cluster_number]` | One coreference cluster, represented as single-token spans. Cluster numbers start from 1. ~~SpanGroup~~ |

## Config and implementation {#config}

The default config is defined by the pipeline component factory and describes
how the component should be configured. You can override its settings via the
`config` argument on [`nlp.add_pipe`](/api/language#add_pipe) or in your
[`config.cfg` for training](/usage/training#config). See the
[model architectures](/api/architectures#coref-architectures) documentation for
details on the architectures and their arguments and hyperparameters.

> #### Example
>
> ```python
> from spacy_experimental.coref.coref_component import DEFAULT_COREF_MODEL
> from spacy_experimental.coref.coref_util import DEFAULT_CLUSTER_PREFIX
> config={
>     "model": DEFAULT_COREF_MODEL,
>     "span_cluster_prefix": DEFAULT_CLUSTER_PREFIX,
> },
> nlp.add_pipe("experimental_coref", config=config)
> ```

| Setting               | Description                                                                                                                              |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `model`               | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. Defaults to [Coref](/api/architectures#Coref). ~~Model~~ |
| `span_cluster_prefix` | The prefix for the keys for clusters saved to `doc.spans`. Defaults to `coref_clusters`. ~~str~~                                         |

## CoreferenceResolver.\_\_init\_\_ {#init tag="method"}

> #### Example
>
> ```python
> # Construction via add_pipe with default model
> coref = nlp.add_pipe("experimental_coref")
>
> # Construction via add_pipe with custom model
> config = {"model": {"@architectures": "my_coref.v1"}}
> coref = nlp.add_pipe("experimental_coref", config=config)
>
> # Construction from class
> from spacy_experimental.coref.coref_component import CoreferenceResolver
> coref = CoreferenceResolver(nlp.vocab, model)
> ```

Create a new pipeline instance. In your application, you would normally use a
shortcut for this and instantiate the component using its string name and
[`nlp.add_pipe`](/api/language#add_pipe).

| Name                  | Description                                                                                         |
| --------------------- | --------------------------------------------------------------------------------------------------- |
| `vocab`               | The shared vocabulary. ~~Vocab~~                                                                    |
| `model`               | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. ~~Model~~           |
| `name`                | String name of the component instance. Used to add entries to the `losses` during training. ~~str~~ |
| _keyword-only_        |                                                                                                     |
| `span_cluster_prefix` | The prefix for the key for saving clusters of spans. ~~bool~~                                       |

## CoreferenceResolver.\_\_call\_\_ {#call tag="method"}

Apply the pipe to one document. The document is modified in place and returned.
This usually happens under the hood when the `nlp` object is called on a text
and all pipeline components are applied to the `Doc` in order. Both
[`__call__`](/api/coref#call) and [`pipe`](/api/coref#pipe) delegate to the
[`predict`](/api/coref#predict) and
[`set_annotations`](/api/coref#set_annotations) methods.

> #### Example
>
> ```python
> doc = nlp("This is a sentence.")
> coref = nlp.add_pipe("experimental_coref")
> # This usually happens under the hood
> processed = coref(doc)
> ```

| Name        | Description                      |
| ----------- | -------------------------------- |
| `doc`       | The document to process. ~~Doc~~ |
| **RETURNS** | The processed document. ~~Doc~~  |

## CoreferenceResolver.pipe {#pipe tag="method"}

Apply the pipe to a stream of documents. This usually happens under the hood
when the `nlp` object is called on a text and all pipeline components are
applied to the `Doc` in order. Both [`__call__`](/api/coref#call) and
[`pipe`](/api/coref#pipe) delegate to the [`predict`](/api/coref#predict) and
[`set_annotations`](/api/coref#set_annotations) methods.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> for doc in coref.pipe(docs, batch_size=50):
>     pass
> ```

| Name           | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| `stream`       | A stream of documents. ~~Iterable[Doc]~~                      |
| _keyword-only_ |                                                               |
| `batch_size`   | The number of documents to buffer. Defaults to `128`. ~~int~~ |
| **YIELDS**     | The processed documents in order. ~~Doc~~                     |

## CoreferenceResolver.initialize {#initialize tag="method"}

Initialize the component for training. `get_examples` should be a function that
returns an iterable of [`Example`](/api/example) objects. **At least one example
should be supplied.** The data examples are used to **initialize the model** of
the component and can either be the full training data or a representative
sample. Initialization includes validating the network,
[inferring missing shapes](https://thinc.ai/docs/usage-models#validation) and
setting up the label scheme based on the data. This method is typically called
by [`Language.initialize`](/api/language#initialize).

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> coref.initialize(lambda: examples, nlp=nlp)
> ```

| Name           | Description                                                                                                                                                                |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_examples` | Function that returns gold-standard annotations in the form of [`Example`](/api/example) objects. Must contain at least one `Example`. ~~Callable[[], Iterable[Example]]~~ |
| _keyword-only_ |                                                                                                                                                                            |
| `nlp`          | The current `nlp` object. Defaults to `None`. ~~Optional[Language]~~                                                                                                       |

## CoreferenceResolver.predict {#predict tag="method"}

Apply the component's model to a batch of [`Doc`](/api/doc) objects, without
modifying them. Clusters are returned as a list of `MentionClusters`, one for
each input `Doc`. A `MentionClusters` instance is just a list of lists of pairs
of `int`s, where each item corresponds to a cluster, and the `int`s correspond
to token indices.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> clusters = coref.predict([doc1, doc2])
> ```

| Name        | Description                                                                  |
| ----------- | ---------------------------------------------------------------------------- |
| `docs`      | The documents to predict. ~~Iterable[Doc]~~                                  |
| **RETURNS** | The predicted coreference clusters for the `docs`. ~~List[MentionClusters]~~ |

## CoreferenceResolver.set_annotations {#set_annotations tag="method"}

Modify a batch of documents, saving coreference clusters in `Doc.spans`.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> clusters = coref.predict([doc1, doc2])
> coref.set_annotations([doc1, doc2], clusters)
> ```

| Name       | Description                                                                  |
| ---------- | ---------------------------------------------------------------------------- |
| `docs`     | The documents to modify. ~~Iterable[Doc]~~                                   |
| `clusters` | The predicted coreference clusters for the `docs`. ~~List[MentionClusters]~~ |

## CoreferenceResolver.update {#update tag="method"}

Learn from a batch of [`Example`](/api/example) objects. Delegates to
[`predict`](/api/coref#predict).

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> optimizer = nlp.initialize()
> losses = coref.update(examples, sgd=optimizer)
> ```

| Name           | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `examples`     | A batch of [`Example`](/api/example) objects to learn from. ~~Iterable[Example]~~                                        |
| _keyword-only_ |                                                                                                                          |
| `drop`         | The dropout rate. ~~float~~                                                                                              |
| `sgd`          | An optimizer. Will be created via [`create_optimizer`](#create_optimizer) if not set. ~~Optional[Optimizer]~~            |
| `losses`       | Optional record of the loss during training. Updated using the component name as the key. ~~Optional[Dict[str, float]]~~ |
| **RETURNS**    | The updated `losses` dictionary. ~~Dict[str, float]~~                                                                    |

## CoreferenceResolver.create_optimizer {#create_optimizer tag="method"}

Create an optimizer for the pipeline component.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> optimizer = coref.create_optimizer()
> ```

| Name        | Description                  |
| ----------- | ---------------------------- |
| **RETURNS** | The optimizer. ~~Optimizer~~ |

## CoreferenceResolver.use_params {#use_params tag="method, contextmanager"}

Modify the pipe's model, to use the given parameter values. At the end of the
context, the original parameters are restored.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> with coref.use_params(optimizer.averages):
>     coref.to_disk("/best_model")
> ```

| Name     | Description                                        |
| -------- | -------------------------------------------------- |
| `params` | The parameter values to use in the model. ~~dict~~ |

## CoreferenceResolver.to_disk {#to_disk tag="method"}

Serialize the pipe to disk.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> coref.to_disk("/path/to/coref")
> ```

| Name           | Description                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path`         | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                                                            |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~                                                |

## CoreferenceResolver.from_disk {#from_disk tag="method"}

Load the pipe from disk. Modifies the object in place and returns it.

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> coref.from_disk("/path/to/coref")
> ```

| Name           | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `path`         | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                 |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~     |
| **RETURNS**    | The modified `CoreferenceResolver` object. ~~CoreferenceResolver~~                              |

## CoreferenceResolver.to_bytes {#to_bytes tag="method"}

> #### Example
>
> ```python
> coref = nlp.add_pipe("experimental_coref")
> coref_bytes = coref.to_bytes()
> ```

Serialize the pipe to a bytestring, including the `KnowledgeBase`.

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The serialized form of the `CoreferenceResolver` object. ~~bytes~~                          |

## CoreferenceResolver.from_bytes {#from_bytes tag="method"}

Load the pipe from a bytestring. Modifies the object in place and returns it.

> #### Example
>
> ```python
> coref_bytes = coref.to_bytes()
> coref = nlp.add_pipe("experimental_coref")
> coref.from_bytes(coref_bytes)
> ```

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `bytes_data`   | The data to load from. ~~bytes~~                                                            |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The `CoreferenceResolver` object. ~~CoreferenceResolver~~                                   |

## Serialization fields {#serialization-fields}

During serialization, spaCy will export several data fields used to restore
different aspects of the object. If needed, you can exclude them from
serialization by passing in the string names via the `exclude` argument.

> #### Example
>
> ```python
> data = coref.to_disk("/path", exclude=["vocab"])
> ```

| Name    | Description                                                    |
| ------- | -------------------------------------------------------------- |
| `vocab` | The shared [`Vocab`](/api/vocab).                              |
| `cfg`   | The config file. You usually don't want to exclude this.       |
| `model` | The binary model data. You usually don't want to exclude this. |

# Transformers.rb

:slightly_smiling_face: State-of-the-art [transformers](https://github.com/huggingface/transformers) for Ruby

[![Build Status](https://github.com/ankane/transformers-ruby/actions/workflows/build.yml/badge.svg)](https://github.com/ankane/transformers-ruby/actions)

## Installation

First, [install Torch.rb](https://github.com/ankane/torch.rb#installation).

Then add this line to your application’s Gemfile:

```ruby
gem "transformers-rb"
```

## Getting Started

- [Models](#models)
- [Pipelines](#pipelines)

## Models

### sentence-transformers/all-MiniLM-L6-v2

[Docs](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)

```ruby
sentences = ["This is an example sentence", "Each sentence is converted"]

model = Transformers::SentenceTransformer.new("sentence-transformers/all-MiniLM-L6-v2")
embeddings = model.encode(sentences)
```

### sentence-transformers/multi-qa-MiniLM-L6-cos-v1

[Docs](https://huggingface.co/sentence-transformers/multi-qa-MiniLM-L6-cos-v1)

```ruby
query = "How many people live in London?"
docs = ["Around 9 Million people live in London", "London is known for its financial district"]

model = Transformers::SentenceTransformer.new("sentence-transformers/multi-qa-MiniLM-L6-cos-v1")
query_embedding = model.encode(query)
doc_embeddings = model.encode(docs)
scores = Torch.mm(Torch.tensor([query_embedding]), Torch.tensor(doc_embeddings).transpose(0, 1))[0].to_a
doc_score_pairs = docs.zip(scores).sort_by { |d, s| -s }
```

### mixedbread-ai/mxbai-embed-large-v1

[Docs](https://huggingface.co/mixedbread-ai/mxbai-embed-large-v1)

```ruby
def transform_query(query)
  "Represent this sentence for searching relevant passages: #{query}"
end

docs = [
  transform_query("puppy"),
  "The dog is barking",
  "The cat is purring"
]

model = Transformers::SentenceTransformer.new("mixedbread-ai/mxbai-embed-large-v1")
embeddings = model.encode(docs)
```

### opensearch-project/opensearch-neural-sparse-encoding-v1

[Docs](https://huggingface.co/opensearch-project/opensearch-neural-sparse-encoding-v1)

```ruby
docs = ["The dog is barking", "The cat is purring", "The bear is growling"]

model_id = "opensearch-project/opensearch-neural-sparse-encoding-v1"
model = Transformers::AutoModelForMaskedLM.from_pretrained(model_id)
tokenizer = Transformers::AutoTokenizer.from_pretrained(model_id)
special_token_ids = tokenizer.special_tokens_map.map { |_, token| tokenizer.vocab[token] }

feature = tokenizer.(docs, padding: true, truncation: true, return_tensors: "pt", return_token_type_ids: false)
output = model.(**feature)[0]

values, _ = Torch.max(output * feature[:attention_mask].unsqueeze(-1), dim: 1)
values = Torch.log(1 + Torch.relu(values))
values[0.., special_token_ids] = 0
embeddings = values.to_a
```

## Pipelines

Named-entity recognition

```ruby
ner = Transformers.pipeline("ner")
ner.("Ruby is a programming language created by Matz")
```

Sentiment analysis

```ruby
classifier = Transformers.pipeline("sentiment-analysis")
classifier.("We are very happy to show you the 🤗 Transformers library.")
```

Question answering

```ruby
qa = Transformers.pipeline("question-answering")
qa.(question: "Who invented Ruby?", context: "Ruby is a programming language created by Matz")
```

Feature extraction

```ruby
extractor = Transformers.pipeline("feature-extraction")
extractor.("We are very happy to show you the 🤗 Transformers library.")
```

Image classification

```ruby
classifier = Transformers.pipeline("image-classification")
classifier.(URI("https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/pipeline-cat-chonk.jpeg"))
```

Image feature extraction

```ruby
extractor = Transformers.pipeline("image-feature-extraction")
extractor.(URI("https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/pipeline-cat-chonk.jpeg"))
```

## API

This library follows the [Transformers Python API](https://huggingface.co/docs/transformers/index). Only a few model architectures are currently supported:

- BERT
- DistilBERT
- ViT

## History

View the [changelog](https://github.com/ankane/transformers-ruby/blob/master/CHANGELOG.md)

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/transformers-ruby/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/transformers-ruby/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features

To get started with development:

```sh
git clone https://github.com/ankane/transformers-ruby.git
cd transformers-ruby
bundle install
bundle exec rake download:files
bundle exec rake test
```

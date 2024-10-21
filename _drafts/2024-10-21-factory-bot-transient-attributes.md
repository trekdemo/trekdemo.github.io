---
categories: [TDD, Testing, FactoryBot]
date: 2024-10-21
layout: post
title: Mi legyen?
---

Today, I saw a pull request where making the test setup explicit enough

I've read [this article](https://read.readwise.io/read/01hnzvbhy3j3kdfy0b5tyz3sts) back in February and it really resonated with me. It gives a tip to create
Since February I haven't reviewed PRs that touched FactoryBot factories.
Today, one came to me. The idea is that we have a localized model that we use to store translated image files and make use of them.
The translations have a few attributes, and at least one translation of one attribute must be present. This creates a somewhat complex challenge for the factory definition.

- [ ] Get a good sample data structure
- [ ] Maybe visualize it.

So in the end we used something like this

The end goal was to go from a long test setup, to a simple but expressive one-liner.

```ruby
RSpec.describe do
  before do
    picture = Rack::Test::UploadedFile.new("sample.jpg", "image/jpg", true)
    manual = Rack::Test::UploadedFile.new("sample.pdf", "application/pdf", true)

    product = create(:product, brand: "General brand")
    product.translations.create(locale: "en", picture:, manual:)
    product.translations.create(locale: "fr", picture:, manual:)
    product.translations.create(locale: "hu", picture:, manual:)
  end
end
```

```ruby
subject(:product) { create(:product, with_translations: [:en, :nl, :hu])}
```

Since a translation is required, the base factory has to create a single translation. However, the content of the translation is not important and rarely checked.

```ruby
# Models
class ProductImage < ApplicationRecord
  translates :picture, :manual
  accepts_nested_attributes_for :translations
end

FactoryBot.define do
  factory :product do
    transient do
      with_translations { %i[en] }
    end

    brand { "ACME corp" }
    price { 10.99 }
    translations_attributes do
      picture = Rack::Test::UploadedFile.new("sample.jpg", "image/jpg", true)
      manual = Rack::Test::UploadedFile.new("sample.pdf", "application/pdf", true)

      with_translations.map do |locale|
        { locale:, picture:, manual: }
      end
    end
end
```

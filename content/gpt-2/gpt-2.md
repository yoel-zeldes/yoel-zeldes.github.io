Title: The Accessibility of GPT-2 - Text Generation and Fine-tuning
Slug: gpt-2
Date: 2019-11-28 23:00
Tags: deep learning, NLP, NLG, GPT-2
Summary: Text generation using GPT-2 is quite easy, using the right tools. Learn how to do it, as well as how to fine-tune the model on your own dataset.
header_cover: images/gpt-2/cover.jpg

Natural Language Generation (NLG) is a well studied subject among the NLP community. With the rise of deep learning methods, NLG has become better and better. Recently, OpenAI has pushed the limits, with the release of [GPT-2](https://openai.com/blog/better-language-models) - a [Transformers](https://arxiv.org/abs/1706.03762) based model that predicts the next [token](https://arxiv.org/abs/1508.07909) at each time space.

Nowadays it’s quite easy to use these models - you don’t need to implement the code yourself, or train the models using expensive resources. HuggingFace, for instance, has [released an API](https://huggingface.co/transformers) that eases the access to the pretrained GPT-2 OpenAI has published. Some of its features include generating text, as well as fine-tuning the model on your own dataset - shifting the learned distribution so that the model will generate text from a new domain.

Doing all of these is easy - it’s only a matter of pip installing the relevant packages and launching a python script. However, to save you the trouble, you could use one of the available platforms such as [Spell](https://spell.run) - you just specify what you want to run, and Spell will take care of the rest (download the code, install the packages, allocate compute resources, manage results).

While not being a Spell advocate (I haven’t even tried other features of the platform, or tried other platforms at all), I decided to write a tutorial that details the process I’ve just described. To find out more, you can find the tutorial [here](https://community.spell.run/hc/en-us/articles/360038909713-GPT-2-text-generation-is-not-something-to-joke-about).

If you also like to play around with machine generated text, feel free to leave a comment with interesting texts you got. :)

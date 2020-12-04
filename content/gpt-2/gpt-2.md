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

---

*UPDATE*: it seems the tutorial is no longer available in the aforementioned link. Although it’s a bit outdated (the hugging face API has changed a lot since then), here is the full text:

<br />

Natural Language Generation (NLG) is a well studied subject among the NLP community. One approach to tackle the challenge of text generation is to factorize the probability of a sequence of tokens (e.g. words or [Byte Pair Encoding](https://arxiv.org/abs/1508.07909)) $P(x_1, \ldots, x_n)$ into the multiplication of the probabilities of getting each of the tokens $x_1$, …, $x_n$ conditioned on the tokens preceding it: $\prod_{t=1}^{n}P(x_t|x_{<t})$. Given a training dataset, one could train such a model to maximize the probability of the next token at each time step. Once the model has been trained, you could generate text by sampling from the distribution one token at a time. Easy as a breeze.

With the rise of deep learning methods, NLG has become better and better. Recently, OpenAI have pushed the limits, with the release of [GPT-2](https://openai.com/blog/better-language-models). This model uses the well known [Transformers architecture](https://arxiv.org/abs/1706.03762): in order to calculate the distribution over the next token, the model simultaneously uses the previous tokens using a self-attention mechanism.

Recently, HuggingFace have [released an API](https://huggingface.co/transformers) easing the access to GPT-2. One of its features is generating text using the pre-trained model:

```bash
spell run --github-url https://github.com/huggingface/transformers.git \
  --pip tqdm \
  --pip boto3 \
  --pip requests \
  --pip regex \
  --pip sacremoses \
  "python examples/run_generation.py \
    --model_type=gpt2 \
    --length=70 \
    --prompt=' ' \
    --model_name_or_path=gpt2"
```
```text
💫 Casting spell #1…
✨ Stop viewing logs with ^C
✨ Machine_Requested… done
✨ Building… done
✨ Run is running
…
…
…
$5.30-$10.00

FREE SHIPPING

Items without a shipping address will be delivered to your confirmation email when you purchase your product.

Use "POOL" when ordering; deliveries to POOL addresses are completely separate from shipping.<|endoftext|>Earth's spin to new low the closer Earth takes to the Sun's
✨ Saving… done
✨ Pushing… done
🎉 Total run time: 1m7.057677s
🎉 Run 1 complete
```

That was easy! OpenAI have used diverse data found on the web for training the model, so the generated text can be pretty much any natural looking text. But what if instead of diversity, we’d like to generate a specific kind of text? Let’s try to generate jokes! To do so, we’ll have to train the model using a dataset of jokes. Unfortunately, getting such a dataset would be ridiculously hard! To train GPT-2, which has 124M weights to be learned (and this is merely the smaller version of the architecture), we need a huge amount of data! But how are we going to get that many jokes? The short answer is: we won’t.

Learning to generate jokes involves learning how to generate natural-looking text, as well as making sure this text is funny. The first part is where most of the learning happens. Using the pre-trained version of GPT-2 as a starting point, the model won’t have to learn how to generate natural-looking text from scratch. All it’ll have to learn is to concentrate the distribution over text that is funny. A relatively small dataset will do for the task.

Don’t get me wrong, the dataset we’ll be using isn’t big enough to meaningfully learn anything useful. Moreover, training a model to generalize the concept of humor is a hard problem. However, for the purpose of this post - learning how to use and fine-tune a model such as GPT-2 - this will do: we’ll witness how the dataset shifts the model’s distribution towards text that looks, to some extent, like jokes.

We’ll use one-liner jokes from [short-jokes-dataset](https://raw.githubusercontent.com/amoudgl/short-jokes-dataset/master/data/onelinefun.csv) to fine-tune GPT-2. Being shorter than the average joke, it’ll be easier for the model to learn their distribution. So first thing’s first, let’s get the data:

```bash
spell run "wget -O data.csv https://raw.githubusercontent.com/amoudgl/short-jokes-dataset/master/data/onelinefun.csv && python -c \"import csv; f_in = open('data.csv', 'r'); f_out = open('data.txt', 'w'); f_out.write('\n'.join(row['Joke'] for row in csv.DictReader(f_in)))\""
```
```text
💫 Casting spell #2…
✨ Stop viewing logs with ^C
✨ Building… done
✨ Machine_Requested… done
✨ Run is running
--2019-11-09 21:36:14--  https://raw.githubusercontent.com/amoudgl/short-jokes-dataset/master/data/onelinefun.csv
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.0.133, 151.101.64.133, 151.101.128.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.0.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 253462 (248K) [text/plain]
Saving to: ‘data.csv’

     0K .......... .......... .......... .......... .......... 20% 3.34M 0s
    50K .......... .......... .......... .......... .......... 40% 6.72M 0s
   100K .......... .......... .......... .......... .......... 60%  167M 0s
   150K .......... .......... .......... .......... .......... 80%  122M 0s
   200K .......... .......... .......... .......... .......   100% 6.55M=0.03s

2019-11-09 21:36:14 (8.14 MB/s) - ‘data.csv’ saved [253462/253462]

✨ Saving… done
✨ Pushing… done
🎉 Total run time: 13.07418s
🎉 Run 2 complete
```

HuggingFace have already provided us with a script to fine-tune GPT-2:

```bash
spell run --github-url https://github.com/huggingface/transformers.git \
  --pip tqdm \
  --pip boto3 \
  --pip requests \
  --pip regex \
  --pip sacremoses \
  -m runs/2/data.txt \
  "python examples/run_lm_finetuning.py \
    --output_dir=output \
    --model_type=gpt2 \
    --model_name_or_path=gpt2 \
    --per_gpu_train_batch_size=2 \
    --num_train_epochs=10 \
    --do_train \
    --train_data_file=data.txt"
```
```text
💫 Casting spell #3…
✨ Stop viewing logs with ^C
✨ Machine_Requested… done
✨ Building… done
✨ Mounting… done
✨ Run is running
…
…
…
🎉 Total run time: 44h36m34.553059s
🎉 Run 3 complete
```

Note that the downloaded data from the previous run is mounted using the `-m` flag.
Even though we’ve used a small dataset (3K examples), running 10 epochs on a CPU took about 44 hours. It only shows how big the model is. This is why you should use a GPU if you want to use a bigger dataset or run many experiments (e.g. tune hyper parameters).

Let’s try to generate a joke, after mounting the result of the previous run:

```bash
spell run --github-url https://github.com/huggingface/transformers.git \
  --pip tqdm \
  --pip boto3 \
  --pip requests \
  --pip regex \
  --pip sacremoses \
  -m runs/3/output \
  "python examples/run_generation.py \
    --model_type=gpt2 \
    --length=70 \
    --prompt=' ' \
    --model_name_or_path=output"
```
```text
💫 Casting spell #4…
✨ Stop viewing logs with ^C
✨ Machine_Requested… done
✨ Building… done
✨ Run is running
…
…
…
"I've got seven fingers! But I don't have those!"
Your childhood might be difficult, but at least it doesn't taste like your grandfather's.
Funny things never happen, in life.
Friends, We've met on the beach. What's wrong with you?
If I'm speaking honestly, I could use some
✨ Saving… done
✨ Pushing… done
🎉 Total run time: 51.047054s
🎉 Run 4 complete
```

The model has learned to generate short sentences, which is typical for our dataset. This relatively easy to grasp data statistic was well learned! Regarding how funny the model is - well… I’ll leave you to judge!

![](images/gpt-2/joke.jpg)

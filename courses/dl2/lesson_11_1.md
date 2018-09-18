# Lesson 11:  Neural Translation

(09-Apr-2018, live)  

- [Wiki Lesson 11](http://forums.fast.ai/t/part-2-lesson-11-wiki/14699)
- [Video Lesson 11](https://www.youtube.com/watch?v=tY0n9OT5_nA&feature=youtu.be) 
  - video length:  2:15:57
- http://course.fast.ai/lessons/lesson11.html
- Notebook:  
   * [translate.ipynb](https://github.com/fastai/fastai/blob/master/courses/dl2/translate.ipynb)

---
# Lesson Description
Today we’re going to learn to **translate French into English**! To do so, we’ll learn how to add **attention to an LSTM** in order to build a sequence to sequence (seq2seq) model. But before we do, we’ll do a review of some key RNN foundations, since a solid understanding of those will be critical to understanding the rest of this lesson.

A seq2seq model is one where both the input and the output are sequences, and can be of difference lengths. Translation is a good example of a seq2seq task. Because each translated word can correspond to one or more words that could be anywhere in the source sentence, we learn an attention mechanism to figure out which words to focus on at each time step. We’ll also learn about some other tricks to improve seq2seq results, including teacher forcing and bidirectional models.

We finish the lesson by discussing the amazing DeVISE paper, which shows how we can bridge the divide between text and images, using them both in the same model!

---
## `00:00` Leslie Smith: super-convergence work
- I want to start pointing out a couple of mini-cool things that happened this week.  One thing I'm really excited about is we briefly talked about how [Leslie Smith](https://twitter.com/lnsmith613) has a new paper out, and it basically, the paper goes, takes his previous two key papers (1: cyclical learning rates and 2: super convergence) and builds on them with a number of experiments to show how you can achieve super-convergence.  Super-convergence lets you train models 5 times faster than previous, kind of step-wise approaches.  It's not 5 times faster than CLR, but it's faster than CLR as well. And the key is that **super-convergence lets you get up to like massively high learning rates** by somewhere between 1 and 3 which is quite amazing.  
- And so the interesting thing about super-convergence is that it... You actually train at those very high learning rates for quite a large percentage of your epochs, and during that time, the loss doesn't really improve very much.  But the trick is it's doing a lot of searching through the space to find really generalizable areas, it seems. 

### `01:25` Sylvain Gugger: super-convergence work
- We kind of had a lot of what we needed in fastai to achieve this, but we're missing a couple of bits.  And so, **[Sylvain Gugger](https://sgugger.github.io)** has done an amazing job of flushing out the pieces that we are missing and then confirming that he has actually achieved super-convergence on training on [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html).  
- I think this is the first time that this has been done that I've heard of, outside of Leslie Smith himself.
- He (Sylain) has got a great blogpost up there [The 1Cycle Policy](https://sgugger.github.io/the-1cycle-policy.html#the-1cycle-policy), which is what Leslie Smith called this approach.
- And this is actually what 1-cycle looks like [upside down V].  It's a single cyclical learning rate but the key difference here is that the going up bit is the same length as the going-down bit, right.  So you go up really slowly and then at the end, for a tenth of the time, you then have this little bit where you go down even further.  And it's interesting.  Obviously, this is a very easy thing to show, a very easy thing to explain.  Sylvain has added it to fastai under the... temporarily it's called the use CLR beta.  By the time you watch this on the video [MOOC], it will probably be called one-cycle, something like that.
- But, you can use this right now.  So that's one key piece to getting these massively high learning rates, and he shows a number of experiments when you do that.
- A second key piece is that as you do this to the learning rate [upside down V], you do this to the momentum [V shape].  So when the learning rate is low, it's fine to have a high momentum but then when the learning rate gets up really high, your momentum needs to be quite a bit lower.  So, this is also part of what he has added to the library, is this cyclical momentum.  
  - And so with these two things, you can train for about a 1/5th of the number of epochs with a stepwise learning rate schedule.  
  - Then you can drop your weight decay down by about 2 orders of magnitude.  
  - You can often remove most or all of your dropout 
- And so you end up with something that is trained faster and generalizes better.
- And it actually turns out Sylvain got quite a bit better accuracy than Leslie Smith's paper.  His guess, I was pleased to see, is because our data augmentation defaults are better than Leslie's.  I hope that's true.  
- So check that out.

## `03:50` Hamel Husain of GitHub: sequence to sequence models
- Another cool thing... as I just said.. there have been so many cool things this week, I'm just going to pick two.
- [Hamel Husain](https://twitter.com/hamelhusain?lang=en) who works at GitHub.
- [Google created a cool demo of my blog post (based on what I learned in Fast.AI)](http://forums.fast.ai/t/google-created-a-cool-demo-of-my-blog-post-based-on-what-i-learned-in-fast-ai/14680)
- [Kubernetes](https://kubernetes.io) is an open-source system for automating deployment, scaling, and management of containerized applications.
- I just really like this:  there's a fairly new project called [Kubeflow](https://www.kubeflow.org) which is basically TensorFlow for Kubernetes.  Hamel wrote a very nice article about magical sequence to sequence models:  [How To Create Data Products That Are Magical Using Sequence-to-Sequence Models](https://towardsdatascience.com/how-to-create-data-products-that-are-magical-using-sequence-to-sequence-models-703f86a231f8), building data products on that and using Kubernetes to kind of put that into production and so forth. 
- He said the Google Kubeflow team created a demo based on what he wrote earlier this year, directly based on the skills he learned in fastai.  And he will be presenting this technique at KDD. KDD is one of the top academic conferences, so I wanted to share this with folks as a motivation to blog, which I think is a great point.  I don't know anybody who goes out and writes a blog and thinks that, you know, probably none of us really think our blog is actually going to be very good, probably nobody's going to read it... And then when people actually do like it and read it, it's with great surprise.  You just go "oh, that's actually something people were interested to read."
```text
hamelsmuHamel HusainApr 9
The Google Kubeflow team created this cool demo: http://gh-demo.kubeflow.org/118

This is based on what I wrote earlier this year: https://towardsdatascience.com/how-to-create-data-products-that-are-magical-using-sequence-to-sequence-models-703f86a231f884, and is directly based upon the skills I learned in fast.ai.

Also, the unexpected benefits of blogging: I will be presenting this technique at KDD conference this year, and am also working on a tutorial on Kaggle-Learn. I just wanted to share this as a motivation for folks to blog, and also share my excitement with, and say thank you to this class and @jeremy.
```
- So, here is the tool where you can summarize github issues using this tool which is now hosted by Google on the kubeflow.org domain
- live demo link is broken: http://gh-demo.kubeflow.org/
- So, I think that's a great story of getting, you know, if Hamel didn't put his work out there, none of this would have happened and yeah, you can check out his post that made it all happen as well.

## `05:35` Sequence to sequence models, Machine Translation
- so talking of the magic of sequence-to-sequence models, let's build one!
- So, we're going to be specifically working on machine translation.  
- Machine translation is something that's been around for a long time.  But, specifically, we are going to look at an approach called neural translation which is using neural networks for translation.  
- And they didn't know... that wasn't really a thing in any kind of meaningful way until a couple of years ago.  And so, thanks to [Chris Manning](https://twitter.com/chrmanning?lang=en) from Stanford for the next 3 slides.
- [in] 2015, Chris pointed out that neural machine translation first appeared properly and it was pretty crappy compared to the statistical machine translation approaches that use kind of classic, like feature engineering and standard NLP kind of approaches of lots of stemming and fiddling around with word frequencies and n-grams and lots of stuff.
- By a year later, it was better than everything else.  This is on a metric called **BLEU**.  We're not going to discuss the metric because it's not a very good metric, and it's not very interesting, but it's what everybody uses.
- **BLEU** (bilingual evaluation understudy) is an algorithm for evaluating the quality of text which has been machine-translated from one natural language to another. Quality is considered to be the correspondence between a machine's output and that of a human: "the closer a machine translation is to a professional human translation, the better it is" – this is the central idea behind BLEU.[1][2] BLEU was one of the first metrics to claim a high correlation with human judgements of quality,[3][4] and remains one of the most popular automated and inexpensive metrics.
- So, that was the BLEU metric [20] as of the time when Chris did this slide.  As of now, it's up here, it's about 30.
- So, we're kind of seeing machine translation starting down the path that we saw starting computer vision object classification in 2012.  I guess, we just surpassed state-of-the-art, and we are now zipping past it at a great rate.
- It's very unlikely that anybody watching this is actually gonna build a machine translation model because you can go to translate.google.com and use it and it works quite well.
- So, why are we learning about machine translation?  Well, the reason we're learning about machine translation is that the general idea of taking some kind of input like a sentence in French and transforming it into some other kind of output with arbitrary length, such as a sentence in English, is a really useful thing to do.  

#### Applications of seq-to-seq (NLP)
- For example, that thing that we just saw that Hamel...github did... **takes github issues and turns them into summaries.** 
- Other examples is **taking videos and turning them into descriptions.**
- Or taking a... well I don't know, I mean like you know basically anything where you are spitting out kind of an arbitrary sized output, very often that's a sentence.  So maybe taking a CT scan and spitting out a radiology report.  This is where you can use seq-to-seq learning. 

---
### `08:35` 4 Big Wins of Neural MT (Machine Translation) [slide]
#### 1. End-to-end training
All parameters are simultaneously optimized to minimize a loss function on the network's output.

#### 2. Distributed representations share strength
Better exploitation of word and phrase similiarities

#### 3. Better exploitation of context
NMT can use a much bigger context - both source and partial target text - to translate more accurately

#### 4. More fluent text generation
Deep learning text generation is much higher quality

### `08:35` 4 Big Wins of Neural MT (Machine Translation) [class notes]
- So the important thing about a neural machine translation is... more slides from Chris.  And generally, seq-to-seq models is that there is no fussing around with heuristics and hackey feature engineering, whatever.  It's end to end training.  We are able to build these distributed representations which are shared by lots of kinds of concepts within a single network.  We are able to use **Long Term State** in the **RNN** so use a lot more context rather than n-gram kind of approaches.  
- And in the end, the text we are generating uses an RNN as well so we can build something that's more fluid. 

- heuristic definition:  A heuristic technique (/hjʊəˈrɪstɪk/; Ancient Greek: εὑρίσκω, "find" or "discover"), often called simply a heuristic, is any approach to problem solving, learning, or discovery that employs a practical method, not guaranteed to be optimal, perfect, logical, or rational, but instead sufficient for reaching an immediate goal. Where finding an optimal solution is impossible or impractical, heuristic methods can be used to speed up the process of finding a satisfactory solution. 

---
### `09:20` BiLSTMs(+Attn) not just for neural MT [slide]
- Part of speech tagging
- Named entity recognition
- Syntactic parsing (constituency and dependency)
- Reading comprehension
- Question answering
- Text summarization
- ...

#### `09:20` BiLSTMs(+Attn) not just for neural MT [class notes]
- We are going to use a bi-directional LSTM with attention, well, actually we're going to use a bi-directional GRU with attention, but basically the same thing.  So you already know about bi-directional recurrent neural network and **attention** we are going to add on top today.  These general ideas you can use for lots of other things as well as Chris points out on slide [above].

### `09:50` BiLSTMs CODE
- So, let's jump into the code which is in the [translate.ipynb](https://github.com/fastai/fastai/blob/master/courses/dl2/translate.ipynb), funnily enough.  
- And so, we are going to try to translate French into English.  And so the basic idea is that we're going to try and make this look as much like a standard neural network approach as possible.  So we are going to need 3 things.  You will remember the 3 things:  
1.  Data
2.  suitable Architecture
3.  suitable Loss Function

### Data
Once you've got those 3 things, you run `fit` and all things going well, you end up with something that solves your problem.  We generally need (x,y) pairs.  Because we need something we can feed into the loss function and say "I took my x-value which was my French sentence and the loss function says it was meant to generate this English sentence [Fr: x, Eng: y] and then you had your predictions which you would then compare and see how good it is. So, therefore, we need lots of these tuples of French sentences with their equivalent English sentence.  That's called a **parallel corpus**.  Obviously, this is harder to find than a corpus for a language model.  Because for a language model, we just need text in some language which you can basically all, for any living language of which the people who use that language like use computers, there will be a few gigabytes at least of text floating around the internet for you to grab. So building a language model is only challenging corpus wise or, you know, ancient languages.  One of our students is trying to do a Sanskrit one, for example, at the moment.  But that's very rarely a problem.  For translation, there are actually some pretty good parallel corpuses available.  For European languages, the European parliament basically has every sentence in every European language.  Anything that goes through the UN is translated to lots of languages.  

#### French to English Corpus
For French to English, we have a particularly nice thing, which is pretty much any semi-official Canadian website will have a French version and an English version.  So, this chap, Chris Callison Burch did a cool thing which is basically to try to transform French urls into English urls by replace "fr" with "en" and hoping that retrieves the the equivalent document.  And then did that for lots and lots of of websites and ended up creating a huge corpus based on millions of web pages. For French to English, we have this *particularly* nice resource.  

We're going to start out by talking about how to **create the data.**  Then, we'll look at the **architecture.**  Then, we'll look at the **loss function.**

#### Bounding Boxes
For **bounding boxes**, all of the interesting stuff was in the **loss function**, but for **neural translation**, all of the interesting stuff is going to be in the **architecture.**

### `13:15` Translation Files
- So, let's zip through this pretty quickly.  And one of the things I want you to think about particularly is, what are the relationships, the similarities in terms of the tasks we're doing and how we do  



---
layout: default
active_tab: homework
img: new_robot_2x.png
img_link: https://xkcd.com/149/
caption: Natural Language Commands
title: CIS 521 Robot Excercise 4 "Commanding Robots with Natural Language" (Extra Credit)
attribution: This homework assignment was developed for UPenn's Artificial Intelligence class (CIS 521) in Fall 2019 John Zhang, Calvin Zhenghua Chen, and Chris Callison-Burch with help from Yrvine Thelusma.
release_date: 2019-11-19
due_date: 2019-12-03 23:59:00EST
submission_link: https://www.gradescope.com/courses/59562
materials:
    - 
      name: r2d2_hw4.zip
      url: r2d2_hw4.zip
readings:
-
   title: Dialogue Systems and Chatbots 
   authors: Dan Jurafsky and James H. Martin
   venue: Speech and Language Processing (3rd edition draft)
   type: textbook
   url: https://web.stanford.edu/~jurafsky/slp3/26.pdf
-
   title: Vector Semantics and Embeddings 
   authors: Dan Jurafsky and James H. Martin
   venue: Speech and Language Processing (3rd edition draft)
   type: textbook
   url: https://web.stanford.edu/~jurafsky/slp3/6.pdf
-
   title: Linguistic Regularities in Continuous Space Word Representations
   authors: Tomas Mikolov, Wen-tau Yih, Geoffrey Zweig
   venue: NACL 2013
   type: conference
   url: https://www.aclweb.org/anthology/N13-1090/
-
   title: Magnitude&colon; A Fast, Efficient Universal Vector Embedding Utility Package
   authors: Ajay Patel, Alexander Sands, Chris Callison-Burch, Marianna Apidianaki
   venue: ACL 2018
   type: conference
   url: https://www.aclweb.org/anthology/D18-2021/
-
   title: Learning to Parse Natural Language Commands to a Robot Control System
   authors: Cynthia Matuszek and Evan Herbst and Luke S. Zettlemoyer and Dieter Fox
   venue: ISER 2012
   type: conference
   url: https://homes.cs.washington.edu/~lsz/papers/mhzf-iser12.pdf
   optional: true
-
   title: Developing Skills for Amazon Alexa
   authors: Amazon
   type: website
   venue: developer tutorial
   url: https://developer.amazon.com/en-US/alexa/alexa-skills-kit
   optional: true
-
   title: Getting Started with Rasa
   authors: Rasa
   type: website
   venue: developer tutorial
   url: https://rasa.com/docs/getting-started/
   optional: true
---

<!-- Check whether the assignment is ready to release -->
{% capture today %}{{'now' | date: '%s'}}{% endcapture %}
{% capture release_date %}{{page.release_date | date: '%s'}}{% endcapture %}
{% if release_date > today %} 
<div class="alert alert-danger">
Warning: this assignment is out of date.  It may still need to be updated for this year's class.  Check with your instructor before you start working on this assignment.
</div>
{% endif %}
<!-- End of check whether the assignment is up to date -->


<!-- Check whether the assignment is up to date -->
{% capture this_year %}{{'now' | date: '%Y'}}{% endcapture %}
{% capture due_year %}{{page.due_date | date: '%Y'}}{% endcapture %}
{% if this_year != due_year %} 
<div class="alert alert-danger">
Warning: this assignment is out of date.  It may still need to be updated for this year's class.  Check with your instructor before you start working on this assignment.
</div>
{% endif %}
<!-- End of check whether the assignment is up to date -->


<div class="alert alert-info">
This assignment is due on {{ page.due_date | date: "%A, %B %-d, %Y" }} before {{ page.due_date | date: "%I:%M%p" }}. 
</div>

{% if page.materials %}
<div class="alert alert-info">
You can download the materials for this assignment here:
<ul>
{% for item in page.materials %}
<li><a href="{{item.url}}">{{ item.name }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}

Robot Excercise 4: Commanding Robots with Natural Language [100 points]
=============================================================

## Setup and Submission

The code for this homework can be found [here](r2d2_hw4.zip). The file you will edit and submit for this homework is `r2d2_hw4.py`.

## Instructions

This assignment will focus on natural language processing (NLP).  NLP is a vibrant subfield of artificial intelligence.  One of the goals of NLP is to allow computers to understand commands spoken in human language.  This enables technologies like Amazon Alexa, Apple’s Siri or Google’s Assistant.

Instead of issuing a command to your droid in Python like

```python
droid.roll(speed=0.5, heading=0, duration=2)
```

we are going to implement an NLP system that will allow you to say

```
"Drive straight ahead for 2 seconds at half speed"
```

Our NLP system will have three main components:

1. __An intent detection module__ that will take in a natural language command, and determine what type of command that a user wants the droid to do.  These will include things like driving commands, light commands, changing the position of its head, making sounds, etc.)
2. __A slot-filler module__ will take the command, and extract the arguments that need to be included when translating the natural language command into its Python equivalent. For example, light comands will need arguments like *which light* to set, and *what color* to change it to.
3. __A speech to text module__ that will allow you to speak into your computer’s microphone and have your voice command converted to text.  For this, we will use an API provided by Google.

<!--
## Background: Word Vectors

Word2vec is a very cool word embedding method that was developed by [Thomas Mikolov et al](https://www.aclweb.org/anthology/N13-1090) in 2013, as part of Google’s NLP team. You can read about it here, in [Chapter 6 of this book](https://web.stanford.edu/~jurafsky/slp3/6.pdf). To summarize: the intuition behind distributional word embeddings like Word2vec is that words that appear in similar contexts have similar meanings. Words that may appear in a 2 word window around burger may include words like delicious, tasty, ate, king, etc., that would identify it with other closely related food items that are also delicious, tasty, and ate, for example. Then, if we wanted to represent a word, we could count how many times a context word appears around it. Say, when crawling over the entirety of Wikipedia, we find that the word "delicious" appears 6 times in total around the word "burger". Then, maybe we could have a 6 in the index for "delicious" in the vector for "burger". Our vectors can get really nasty with a vocabulary of size 10,000, not to mention that they would be very sparse as well.

One of the ways around this is to first fix a size for the word vectors, initialize random values, and then push the vector representations of similar words together, using gradient descent to minimize some sort of distance function. More can be read about the methods the Word2vec strategy uses here: [Chapter 6 of this book](https://web.stanford.edu/~jurafsky/slp3/6.pdf). Word2vec provides small, fixed-dimensional vector embeddings of words, trained on a corpus of Google News articles which contained about 100 billion words.

One of the noteworthy things about the method is that it can be used to solve word analogy problems like:

<p align="center">
man is to king as woman is to [blank]
 </p>
 The way that it they take the vectors representing *king*, *man* and *woman* and perform some vector arithmetic to produce a vector that is close to the expected answer:
  <p align="center">
 $king−man+woman \approx queen$. 
 </p>.


Distributional word embeddings like Word2vec can run into the issue where antonyms, which have very similar contexts, map onto similar vectors: i.e., the similarity between "south" and "north" is 0.967.

One of the nice things about antonyms matching together though is that the word2vec vectors have a good idea what kind of thing you want them to do. For example, north and south are both cardinal directions, and kick and punch have a good similarity score. We will try to leverage this fact to match R2D2 commands to the category of commands they belong to.

## Getting Started with Magnitude and Downloading data

To get accustomed with word2vec, you will play around with the [Magnitude](https://github.com/plasticityai/magnitude)  library.  You will use Magnitude to load a vector model trained using word2vec, and use it to manipulate and analyze the vectors. Please refer [here](https://github.com/plasticityai/magnitude#installation) for the installation guidelines. 

In order to proceed further, you need to use the Medium Google-word2vec embedding model trained on Google News by using file `GoogleNews-vectors-negative300.magnitude` on eniac in `/home1/c/cis530/hw4_2019/vectors/`. *WARNING, THIS FILE IS VERY LARGE, ~5GB
. MAKE SURE YOU HAVE ENOUGH SPACE BEFORE DOWNLOADING*

Once the file is downloaded, refer to the [Using the Libary](https://github.com/plasticityai/magnitude#using-the-library) section and the [Querying](https://github.com/plasticityai/magnitude#querying) section to see how to import and use the methods found in the library.

## Questions about Magnitude [10 points]

1.  What is the dimensionality of these word embeddings? Provide an integer answer.
2.  What are the top-5 most similar words to `couch` (not including `couch` itself)?
3.  According to the word embeddings, which of these words is not like the others? `['dodge_charger', 'ford_taurus', 'honda', 'lamborghini', 'tesla']`
4.  Solve the following analogy: `american` is to `dollar` as `japanese` is to x.

We have provided a file called `part2.txt` for you to submit answers to the questions above.

-->

## 1. Natural Language Commands for R2D2 [15 points]

We're going to begin this assignment by brainstorming different commands that we might like to give to our robot.  We'll take several factors into account:
1. What actions can the robot perform?
2. What are different ways that we can describe those actions?

The type of actions that our R2D2s can perform are dictated by its Python API.  You can see a list of the commands in the API like this:

```python
from client import DroidClient
droid = DroidClient() 
droid.scan() 
droid.connect_to_droid('D2-55A2') # Replace D2-55A2 with your droid's ID
help(droid)
````

Let's put the API commands that `help` lists into different groups.  We'll also list natural language commands that might be associated with each group.  For the first part of this assignment, you will brainstrom 10 unique language commands for each group.  You will submit your sentences along with your code.

<div class="container-fluid">
<div class="row">
<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Driving API

```python
enter_drive_mode(self)
roll(self, speed, angle, time)
turn(self, angle, **kwargs)
update_position_vector(self, speed, angle, time)
roll_time(self, speed, angle, time, **kwargs)
roll_continuous(self, speed, angle, **kwargs)
restart_continuous_roll(self)
stop_roll(self, **kwargs) 
```
</div>

<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Driving sentences

```python
driving_sentences = [
"Go forward for 2 feet, then turn right.",
"North is at heading 50 degrees.",
"Go North.",
"Go East.",
"Go South-by-southeast",
"Run away!",
"Turn to heading 30 degrees.",
"Reset your heading to 0",
"Turn to face North.",
"Start rolling forward.",
"Increase your speed by 50%.",
"Turn to your right.",
"Stop.",
"Set speed to be 0.",
"Set speed to be 20%",
"Turn around", ]
```
</div>




<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Lights API

```python
set_back_LED_color(self, r, g, b)
set_front_LED_color(self, r, g, b)
set_holo_projector_intensity(self, intensity)
set_logic_display_intensity(self, intensity)
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Light sentences

```python
light_sentences = [
"Change the intensity on the holoemitter to maximum.",
"Turn off the holoemitter.",
"Blink your logic display.",
"Change the back LED to green.",
"Turn your back light green.",
"Dim your lights holoemitter.",
"Turn off all your lights.",
"Lights out.",
"Set the RGB values on your lights to be 255,0,0.",
"Add 100 to the red value of your front LED.",
"Increase the blue value of your back LED by 50%.",
"Display the following colors for 2 seconds each: red, orange, yellow, green, blue, purple.",
"Change the color on both LEDs to be green.", ]
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Head API

```python
rotate_head(self, angle)
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Head sentences

```python
head_sentences = [
"Turn your head to face forward.",
"Look behind you.", ]
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Variables about the droid's state


```python
angle = 0
awake = False
back_LED_color = (0, 0, 0)
battery(self)
connected_to_droid = False
continuous_roll_timer = None
drive_mode = False
drive_mode_angle = None
drive_mode_shift = None
drive_mode_spreed = None
front_LED_color = (0, 0, 0)
holo_projector_intensity = 0
logic_display_intensity = 0
stance = 2
waddling = False
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Questions about variables 

```python
state_sentences = [
"What color is your front light?",
"Tell me what color your front light is set to.",
"Is your logic display on?",
"What is your stance?"
"What is your orientation?",
"What direction are you facing?",
"Are you standing on 2 feet or 3?",
"What is your current heading?",
"How much battery do you have left?",
"What is your battery status?",
"Are you driving right now?",
"How fast are you going?",
"What is your current speed?",
"Is your back light red?",
"Are you awake?", ]
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Connection API

```python
connect_to_R2D2(self)
connect_to_R2Q5(self)
disconnect(self)
scan(self)
exit(self)
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Connection sentences

```python
connection_sentences = [
"Connect D2-55A2 to the server",
"Are there any other droids nearby?",
"Disconnect.",
"Disconnect from the server.", ]
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Stance API

```python
set_stance(self, stance, **kwargs)
set_waddle(self, waddle)
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Stance sentences

```python
stance_sentences = [
"Set your stance to be biped.",
"Put down your third wheel.",
"Stand on your tiptoes.",]
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Animations and sounds API

```python
animate(self, i, wait=3)
play_sound(self, soundID, wait=4)
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Animation sentences

```python
animation_sentences = [
"Fall over",
"Scream",
"Make some noise",
"Laugh",
"Play an alarm",]
```
</div>



<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Navigation on a grid
The following grid navigation commands are from the [Droid navigation assignment](hw2/homework2.html), not the provided API.  We will support grid navigation commands too.

```python 
Graph(vertics, edges)
A_star(G, start, goal)
path2move(path)
```
</div>


<div class="col-lg-6 col-md-6 col-xs-12" markdown="1">
### Navigation on a grid

```python 
grid_sentences = [
"You are on a 4 by 5 grid.",
"Each square is 1 foot large.",
"You are at position (0,0).",
"Go to position (3,3).",
"There is an obstacle at position 2,1.",
"There is a chair at position 3,3",
"Go to the left of the chair.",
"It’s not possible to go from 2,2 to 2,3.", ]
```
</div>

</div>
</div>

For each of the 8 categories of commands please create 10 unique sentences on how you might tell the robot to execute one or more of the actions in that category. You can add add your sentence lists to the code by adding them as arrays called `my_driving_sentences`, `my_light_sentences`, `my_head_sentences`, `my_state_sentences`, `my_connection_sentences`, `my_stance_sentences`, `my_animation_sentences`, and `my_grid_sentences`.

One of the amazing thing about language is that there are many different ways of communicating the same intent.  For example, if we wanted to have our R2D2 start waddling, we could say 
```python 
"waddle",
"totter",
"todder",
"teater",
"wobble",
"start to waddle"
"start waddling",
"begin waddling",
"set your stance to waddle",
"try to stand on your tiptoes",
"move up and down on your toes",
"rock from side to side on your toes",
"imitate a duck's walk",
"walk like a duck"
```
Similarly, if we wanted it to stop, we could prefix the command above with a bunch of ways of saying stop:
```python 
"stop your waddle",
"end your waddle",
"don't waddle anymore",
"stop waddling",
"cease waddling",
"stop standing on your toes",
"stand still"
"stop acting like a duck",
"don't walk like a duck",
"stop teetering like that"
"put your feet flat on the ground"
```

The goal of this part of the assignment is to enumerate as many ways of saying a command as you can think of (minimum of 10 per command group).  We will use these to train an intent detection module. 



## 2. Intent Detection [70 points]

In this section, we will take in a new sentence that we have never seen before and try to classify what type of command the user wants to have the the robot execute.  To do so, we will measure the similarity of the user's new sentence with each of our training sentences.  We know what command group each of our training sentences belongs to, so we will find the nearest command sentences to the new sentence, and use their labels as the label of the new sentence.  This is called $k$-nearest neighbor classification.   The label that we will assign will be `driving`, `light`, `head`, `state`, `connection`, `stance`, `animation`, or `grid`.

To calculate how similar two sentences are, we are going to leverage word embeddings that we dicussed in lecture (and that are described in the [Vector Semantics and Embeddings chapter of the Jurafsky and Martin textbook](http://web.stanford.edu/~jurafsky/slp3/6.pdf)).  We will use pre-trained word2vec embeddings, and use the Magnitude python package to work with these embeddings. Then, we will use the embeddings for the words in a sentence to create sentence embeddings.

<!--
1. **[5 points]** Write a function `sentenceToWords(sentence)` that returns a list of the words in a sentence, given a string `sentence` as input. The words in the list should all be **lower case**. Note that some words commonly have punctuation marks inside them, such as “accident-prone”. Our function should treat hyphenated words as **one** word. However, when passing in sentences you can assume that hyphenated words will come in the form of “accident_prone”, where an underscore separates the word instead. There is also one more case where punctuation can be “inside” a word. Your function should work like so:

    ```python
    >>> sentenceToWords("Due to his limp, Jack is accident_prone.")
    ['due', 'to', 'his', 'limp', 'jack', 'is', 'accident_prone']
    >>> sentenceToWords("REALLY?!")
    ['really']
    ```
-->
1. **[5 points]** Write a tokenization function `tokenize(text)` which takes as input a string of text and returns a list of tokens derived from that text. Here, we define a token to be a contiguous sequence of non-whitespace characters.  We will remove punctuation marks and convert the text to lowercase. *Hint: Use the built-in constant `string.punctuation`, found in the `string` module, and/or python's regex library, `re`.*
    
    ```python
    >>> tokenize("  This is an example.  ")
    ['this', 'is', 'an', 'example' ]
    ```
    
    ```python
    >>> tokenize("'Medium-rare,' she said.")
    ['medium', 'rare', 'she', 'said']
    ```

2. **[5 points]** Implement the cosine similarity fuction to compute how similar two vectors are. Here is the mathmatical definition of the different parts of the cosine function.  The __dot product__ between two vectors $$\vec{v}$$ and $$\vec{w}$$ is:
    
    $$\text{dot-product}(\vec{v}, \vec{w}) = \vec{v} \cdot \vec{w} = \sum_{i=1}^{N}{v_iw_i} = v_1w_1 +v_2w_2 +...+v_Nw_N$$
  
    The __vector length__ of a vector $$\vec{v}$$  is defined as:

    $$\|\vec{v}\| = \sqrt{\sum_{i=1}^{N}{v_i^2}}$$

    The __cosine__ of the angles between  $$\vec{v}$$  and $$\vec{w}$$ is:
    
    $$cos \Theta = \frac{\vec{v} \cdot \vec{w}}{\|\vec{v}\|\|\vec{w}\|}$$

    Here $$\Theta$$ represents the angle between $$\vec{v}$$ and $$\vec{w}$$.
    
    Implement a cosine similarity function `cosineSimilarity(vector1, vector2)`, where `vector1` and `vector2` are [numpy arrays](https://docs.scipy.org/doc/numpy/user/quickstart.html).  Your function should return the cosine of the angles between them. You are welcome to use any of [the](https://docs.scipy.org/doc/numpy/reference/generated/numpy.dot.html#numpy.dot) [built-in](https://docs.scipy.org/doc/numpy/reference/generated/numpy.sum.html#numpy.sumsum) [numpy](https://docs.scipy.org/doc/numpy/reference/generated/numpy.square.html) [functions](https://docs.scipy.org/doc/numpy/reference/generated/numpy.sqrt.html)[.](https://docs.scipy.org/doc/numpy/reference/generated/numpy.linalg.norm.html) If you don't have numpy installed on your computer already, you should run `pip install numpy`.

    Here are some examples of what your method should output:

    ```python
    import numpy as np 
    ```

    ```python
    >>> cosineSimilarity(np.array([2, 0]), np.array([0, 1])) 
    0.0
    
    >>> cosineSimilarity(np.array([1, 1]), np.array([1, 1]))
    0.9999999999999998. # It's actually 1.0, but this is close enough.  

    >>> cosineSimilarity(np.array([10, 1]), np.array([1, 10]))
    0.19801980198019803

    >>> v1 = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
    >>> v2 = np.array([9, 8, 7, 6, 5, 4, 3, 2, 1, 0])
    >>> cosineSimilarity(v1, v2)
    0.4210526315789473
    ```


3. **[10 points]** Next, we're going to use word vectors to compute the similarity of sentences.  For this part, we'll use the [Magnitude package](https://github.com/plasticityai/magnitude), which is a fast, efficient Python package for manipulating pre-trained word embeddings.  It was written by former Penn students Ajay Patel and Alex Sands.  You can install it with pip by typing this command into your terminal:
```bash
pip3 install pymagnitude
```
Next, you'll need to download a pre-trained set of word embeddings.  We'll get a set trained with Google's word2vec algorithm, which we discussed in class.  You can download them by clicking on [this link](http://magnitude.plasticity.ai/word2vec/medium/GoogleNews-vectors-negative300.magnitude) or by using this command in your terminal:
```bash
wget http://magnitude.plasticity.ai/word2vec/medium/GoogleNews-vectors-negative300.magnitude
```
**Warning:** the file is very large (5GB).  If you'd like to experiment with another set of word vectors that is smaller, you can [download these GloVE embeddings](http://magnitude.plasticity.ai/glove/heavy/glove.6B.300d.magnitude) which are only 1.4GB.

   After the file downloads, you can access the vectors like this:

   ```python
   from pymagnitude import *
   path = '/Users/ccb/Downloads/' # Change this to where you downloaded the file.
   vectors = Magnitude(path + "GoogleNews-vectors-negative300.magnitude") 
   v = vectors.query("cat") # vector representing the word 'cat'
   w = vectors.query("dog") # vector representing the word 'dog'

   # calculate the cosine similarity with your implementation
   sim = cosineSimilarity(v, w) 
   print(sim)
   ```
   If you implemented the cosine similarity function properly, and if you loaded the vectors from the `GoogleNews-vectors-negative300.magnitude` file, you should get **0.76094574**. If you loaded the vectors from the `glove.6B.300d.magnitude` file you should get **0.6816747**.

   **[0 points]** In the `WordEmbeddings` class, write an initialization method `__init__(self, file_path)` that creates a `Magnitude` object from the path specified by the input, and stores it internally for future use.

   **[10 points]** Implement the function `calcSentenceEmbeddingBaseline(self, sentence)` in the `WordEmbeddings` class that takes in a sentence and returns a vector embedding for that sentence, using the `Magnitude` object stored in your initialization method. If the sentence has no words, you should return a vector of all zeros with the same number of dimensions as a word in the Magnitude vectors.

   For `calcSentenceEmbeddingBaseline(self, sentence)` you should return a component-wise addition of all of the vectors.  All the word vectors will be equal in length.  You will return a sentence vector that is also that length.  The first component of your sentence vector will be the addition of the the first component of each of the words.   Easy right?

   Here's an example of the output you would get
   ```python
   >>> X = WordEmbeddings("/Users/ccb/Downloads/GoogleNews-vectors-negative300.magnitude") # Change this to where you downloaded the file.
   >>> svec1 = X.calcSentenceEmbeddingBaseline("drive forward")
   >>> svec2 = X.calcSentenceEmbeddingBaseline("roll ahead")
   >>> svec3 = X.calcSentenceEmbeddingBaseline("set your lights to purple")
   >>> svec4 = X.calcSentenceEmbeddingBaseline("turn your lights to be blue")
   >>> cosineSimilarity(svec1, svec2)
   0.4255210604304939
   >>> cosineSimilarity(svec1, svec3)
   0.20958250895677447
   >>> cosineSimilarity(svec1, svec4)
   0.30474097280886364
   >>> cosineSimilarity(svec2, svec3)
   0.24962558300148688
   >>> cosineSimilarity(svec2, svec4)
   0.27946534951158214
   >>> cosineSimilarity(svec3, svec4)
   0.8081137933660256
   ```



   The baseline sentence embedding method assumes that all the words in the sentence have the same importance.  
   <!--
   Later on you can implement your own `calcSentenceEmbedding(sentence, vectors)` function that does something more sophisticated if you like.
   -->




4. **[10 points]** We have provided a txt file of training sentences for the R2D2s in a file named r2d2TrainingSentences.txt, as well as a function, `loadTrainingSentences(file_path)`, which reads the file and returns a dictionary with keys `[category]` which map to a list of the sentences belonging to that category.

    ```python
    >>> trainingSentences = loadTrainingSentences("data/r2d2TrainingSentences.txt")
    >>> trainingSentences['animation']
    ['make a gesture', 'speak', 'Play sound number 3.', 'Stop dancing.', 'set off your alarm', ... ]
    ```

    In the `WordEmbeddings` class, write a function `sentenceToEmbeddings(self, commandTypeToSentences)` that converts every sentence in the dictionary returned by `loadTrainingSentences(file_path)` to an embedding. You should return a tuple of two elements. The first element is an m by n numpy array, where m is the number of sentences and n is the length of the vector embedding. Row i of the array should contain the embedding for sentence i. The second element is a dictionary mapping from the index of the sentence to a tuple where the first element is the original sentence, and the second element is a category, such as “driving”. The order of the indices does not matter, but the indices of the matrix and the dictionary should match i.e., sentence j should have an embedding in the jth row of the matrix, and should have itself and its category mapped onto by key j in the dictionary.
    
    ```python
    >>> trainingSentences = loadTrainingSentences("data/r2d2TrainingSentences.txt")
    >>> X = WordEmbeddings("/Users/ccb/Downloads/GoogleNews-vectors-negative300.magnitude") # Change this to where you downloaded the file.
    >>> sentenceEmbeddings, indexToSentence = X.sentenceToEmbeddings(trainingSentences)
    >>> sentenceEmbeddings[14:]
    array([[ 0.08058001,  0.21676847, -0.06604716, ..., -0.03767369,
             0.15602297, -0.07835222],
           [ 0.0350151 ,  0.07319701,  0.0894349 , ..., -0.0442058 ,
            -0.0910254 ,  0.00273301],
           [-0.04938643,  0.2280895 ,  0.24541894, ..., -0.16277108,
             0.00430368, -0.45862231],
           ...,
           [ 0.25111231,  0.33453143,  0.18835592, ..., -0.05870331,
            -0.02659047, -0.47405607],
           [ 0.20804575,  0.07450815,  0.21608222, ...,  0.09974989,
             0.38724095, -0.41757214],
           [ 0.1263006 , -0.2014823 ,  0.17403649, ..., -0.01363612,
            -0.1347626 ,  0.0201975 ]])
    >>> indexToSentence[239]
    ('Turn to heading 50 degrees.', 'driving')
    ```

5. **[10 points]** Now, given an arbitrary input sentence, and an m by n matrix of sentence embeddings, write a function `closestSentence(self, sentence, sentenceEmbeddings)` that returns the index of the closest sentence to the input. This should be the row vector which is closest to the sentence vector of the input. Depending on the indices of your implementation of `sentenceToEmbeddings(self, commandTypeToSentences)`, the following output may vary.

    ```python
    >>> sentenceEmbeddings, _ = X.sentenceToEmbeddings(loadTrainingSentences("data/r2d2TrainingSentences.txt"))
    >>> X.closestSentence("Lights on.", sentenceEmbeddings)
    301
    ```

6. **[25 points]** Now, given an arbitrary input sentence, and a file path to r2d2 commands, write a function `getCategory(self, sentence, file_path)` that returns the category that that sentence should belong to. You should also map sentences that don’t really fit into any of the categories to a new category, “no”, and return “no” if the input sentence does not really fit into any of the categories.

    Simply finding the closest sentence and outputting that category may not be enough for this function. We suggest trying out a k-nearest neighbors approach, and scoring the neighbors in some way to find which category is the best fit. You can write new helper functions to help out. Also, which kind of words appear in almost all sentences and so are not a good way to distinguish between sentence meanings?
        
    ```python
    >>> X.getCategory("Turn your lights green.", "data/r2d2TrainingSentences.txt")
    'light'
    >>> X.getCategory("Drive forward for two feet.", "data/r2d2TrainingSentences.txt")
    'driving'
    >>> X.getCategory("Do not laugh at me.", "data/r2d2TrainingSentences.txt")
    'no'
    ```

    Your implementation for this function can be as free as you want. We will test your function on a test set of sentences. Our training set will be ` r2d2TrainingSentences.txt `, and our test set will be similar to the development set called `r2d2DevelopmentSentences.txt` which we have provided for testing your implementation locally (however, there will be differences, so try not to overfit!). Your accuracy will be compared to scores which we believe are relatively achievable. Anything greater than or equal to a 90% accuracy on the test set will receive a 100%, and anything lower than a 80% accuracy will receive no partial credit. To encourage friendly competition, we have also set up a leaderboard so that you can see how well you are doing against peers.

    When testing this function, we will be passing the same `file_path` over and over again to `getCategory`. We do not want our function to have to calculate the sentence embeddings of sentences in `file_path` repeatedly. Thus, modify `getCategory` so that we do not have to perform repeat operations when passing in the same `file_path` (you can change the `__init__` function as well). We will test this by setting strict runtime bounds for our `getCategory` and `accuracy` tests.

    **[5 points]** To help you with your implementation of `getCategory`, we require that you fill out the code stub for `accuracy(self, training_file_path, dev_file_path)`. This function should test your implementation of `getCategory` faithfully using paths to training and development sets as input. Don't worry about the efficiency of this function! Located in the `data` folder is a development set `r2d2DevelopmentSentences.txt` which we have provided for testing your implementation of `getCategory` locally.

    ```python
    >>> X.accuracy("data/r2d2TrainingSentences.txt", "data/r2d2DevelopmentSentences.txt")
    0.9041095890410958
    ```

    **Note** Before you submit, you need to indicate which Magnitude file you decide to use for your `getCategory` function. If you decide on using the Google Word2Vec vectors, change the `magnitudeFile` variable at the beginning of Section 2 to `"google"`. If you decide that you like the GloVE vectors better, change the `magnitudeFile` variable to `"glove"`. Doing so is **very important**, as this may change how accurate your `getCategory` function is.

<!--
    Your implementation for `getCategory` can be as free as you want. We will test your function on a test set of sentences. Our training set will be ` r2d2TrainingSentences.txt `, and our test set will be similar to the development set `r2d2DevelopmentSentences.txt` (however, there will be differences, so try not to overfit!). Your accuracy will be compared to scores which we believe are relatively achievable. Anything greater than or equal to a 75% accuracy on the test set will receive a 100%, and anything lower than a 60% accuracy will receive no partial credit. To encourage friendly competition, we have also set up a leaderboard so that you can see how well you are doing against peers.
-->

<!--
*For Extra Extra Credit*

Take a look at this [online service](https://github.com/hanxiao/bert-as-service) which uses BERT. [BERT](https://arxiv.org/pdf/1810.04805.pdf) is one of the latest breakthroughs in NLP, and has broken previous state-of-the-art records on a number of tasks. With BERT, even without fine-tuning, you should easily be able to achieve a 0.90 accuracy on our r2d2 test set. If you use BERT in your intent detection function `getCategory(sentence, file_path)`, either with Hanxiao's online service or in some other manner, we will manually give you extra extra credit.
-->


<!--
## 3. How good is your intent detection [5 points]

TODO - write an accuracy function, evaluate your intent detection module on a test set.
-->


## 3. Slot filling [15 points]

Now that we have a good idea which categories our commands belong to, we have to find a way to convert these commands to actions. This can be done via slot-filling, which fills slots in the natural language command corresponding to important values. For example, given the slots NAME, RESTAURANT, TIME and HAS_RESERVED, and a command to a chat-bot such as "John wants to go to Olive Garden", the chat-bot should fill out the slots with values: {NAME: John, RESTAURANT: Olive Garden, TIME: N/A, HAS_RESERVED: False}, and then it can decide to either execute the command or ask for more information given the slot-values.

1. **[15 points]** Using regex or word2vec vectors, populate the functions `lightParser(self, command)` and `drivingParser(self, command)` in the `WordEmbeddings` class to perform slot-filling for the predefined slots, given string input `command`. We will test these functions and give you full credit if you get above a 50% accuracy. These functions do not have to be perfect, but the better these functions are, the better your R2D2 will respond to your commands.

    For `lightParser`, the `lights` slot refers to which lights the command refers to. It is a list with a combination of the strings `"front"`, `"back"`, `"holoEmit"`, and `"logDisp"`, corresponding to whether you believe the command refers to the front light, back light, holoemitter, or logic display, respectively. The order of these values does not matter. If the command wants to increase an RGB value, set the `add` slot to true. If it wants to decrease an RGB value, set the `sub` slot to true. The `on` and `off` fields correspond to whether the lights should be turned on or off (if the command is just changing the color of one of the RGB lights, these slots should not be changed), and should also respond to words like "maximum."

    For `drivingParser`, `add` and `sub` correspond to whether the command wants you to increase/decrease the speed. This should be similar to the add and sub slots from the above `lightParser`, except here you should also make sure these slots respond to words such as `faster` and `slower`. The `directions` slot corresponds to an ordered list of all direction relevant words. Values in the directions list can only be one of `"go"`, `"turn"`, `"by"`, all cardinal directions (`"north"`, `"southwest"`, etc.), and relative directions (out of `"forward"`, `"back"`, `"left"`, and `"right"`). Cardinal combinations found in the command, such as "South-by-southeast", should be parsed to the `directions` slot as `[..., "south", "by", "southeast",...]`. The `"go"` value in this list should also respond to word like `"drive"` and `"roll"`, but be careful when using Word Embeddings here. ( What other value in the directions slot can be confused with `"go"` if using cosine similarity? Is regex a better idea here? )

    All string values should be lower-case.

    Your functions should work like so:

    ```python
    >>> X.lightParser("Set your lights to maximum")
    {'lights': [], 'add': False, 'sub': False, 'on': True, 'off': False}
    >>> X.lightParser("Increase the red RGB value of your front light by 50.")
    {'lights': ['front'], 'add': True, 'sub': False, 'on': False, 'off': False}
    >>> X.lightParser("Turn your holoemitter on.")
    {'lights': ['holoEmit'], 'add': False, 'sub': False, 'on': True, 'off': False}
    ```
   
    ```python
    >>> X.drivingParser("Make your speed faster.")
    {'add': True, 'sub': False, 'directions': []}
    >>> X.drivingParser("Decrease your speed by 50%.")
    {'add': False, 'sub': True, 'directions': ["by"]}
    >>> X.drivingParser("Drive north-by-northwest.")
    {'add': False, 'sub': False, 'directions': ['go', 'north', 'by', 'northwest']}
    >>> X.drivingParser("Go forward for 2 seconds, then turn right.")
    {'add': False, 'sub': False, 'directions': ['go', 'forward', 'turn', 'right']}
    ```

## 4. Try it out!

Now that you are finished with the intent detection and slot filling sections, you can now use the code you have written to talk to your R2D2. Perform the R2D2 server setup instructions found in previous R2D2 homeworks, and move all your files over to your `sphero-project/src` directory. Then, change the ID in line 15 of `robot_com.py` to the ID of your robot, the path on line 16 of r2d2_commands.py to the path of your Magnitude file of choice (from this new directory), and on the command line run `python3 robot_com.py`.

Try out commands like:

```python
"Change your lights to red, periwinkle, azure, green, and magenta."
```

Have fun! Try not to be too mean to your robot :). ( Do not add sentences to the training data if you have not already finished the above sections, as this may change the local behavior of `getCategory`. If you find sentences here that are parsed wrongly, feel free to add them to your example sentences as well! )


## 5. Voice Input [Extra Extra Credit: 15 points]

*For More Extra Extra Credit* Integrate Google Cloud Platform speech-to-text module so that you can command your robot using voice!

To run audio IO, you will need to install portaudio and pyaudio:

```
brew install portaudio
pip3 install pyaudio
```

Next, you need to sign up for a [Google Cloud Platform (GCP) account](https://cloud.google.com/gcp/). When you register a new account, you'll get 300 dollars of free credits. You have to enter your credit card information to sign up, but you will not be billed unless you exceed the 300 dollars limit, so make sure you keep your account information secure! (You may need to use a non-upenn Google account for this part.)

To enable the speech-to-text API, type 'speech' in the search bar, and select "Cloud Speech-to-Text API" from the drop-down menu. Click to enable the API, then click on the "Create Credentials" button. Select the "Cloud Speech-to-Text API" (you do not need the App Engine API), and when setting up roles, make yourself the role administrator. Then, you should be able to get a service account key file (this is going to be in .json format). Rename it `credentials.json` and put it under the `sphero-project/src` folder.

You may also need to install and set up Google Cloud SDK locally. To do this, follow the instructions located in the Quickstart documentation [here](https://cloud.google.com/sdk/docs/quickstarts). Make sure to follow all steps until you are finished with the `Initialize the SDK` section. If during initialization you are asked which project to choose, choose the project that contains the API for speech-to-text.

Then, just install the Python client for the Google API, using:

```
pip3 install google-cloud-speech
```

Now, you should be able to run voice IO on your robot. As before, change the robot serial ID with your own in audio_io.py. Setup the R2D2 server as in previous homeworks, cd into the `sphero-project/src` folder in another Terminal and type:

```
python3 audio_io.py
```

**Note** Depending on how you set up the SDK, you may need to run the following on the command line:

```
export GOOGLE_APPLICATION_CREDENTIALS="/[Path to sphero-project/src]/credentials.json"
```

before you can run audio_io.py.

Notes:
1. If you want to try audio IO, please try command line IO first.

2. If you are able to successfully run audio_io.py, say your command (using voice!) and see if text appears in the Terminal. To end the session, simply say any sentence containing one of the following keywords: "exit", "quit", "bye" or "goodbye".

To receive extra credit for this portion, please come into office hours and show off your project to us!



## Recommended readings

<table>
   {% for publication in page.readings %}
    <tr>
      <td>
  {% if publication.url %}
    <a href="{{ publication.url }}">{{ publication.title }}.</a>
        {% else %}
    {{ publication.title }}.
  {% endif %}
  {{ publication.authors }}.
  {{ publication.venue }}.
</td></tr>
  {% endfor %}
</table>

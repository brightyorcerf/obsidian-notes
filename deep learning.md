#### How LLMs actually work (blog)

(without the boring math stuff)

Most LLMs share the same transformer-family skeleton. The differences come from what each was trained on, scale, configuration, and post-training done on top. Understanding the transformer machinery gets you most of the way there.


![[2.png]]


##### Tokenization

Takes your string and converts it into sequence of integers, where each points to an entry in a fixed vocabulary. Modern LLMs vocab usually contain 10k-100k entries. 

Token ID is the int model uses for vocab entry. Model only works with number not the word. 
Tokens aren't usually words. ex) ex) run + ning = running 
Fun fact: this is why LLMs were infamously bad guessing the number of r's in strawberry. The models do not operate on letters, only token IDs that spell out a word. 

![[3.png]]

Different model families use different tokenizers. GPT models use Byte Pair Encoding variants. The choices affects compute and for things like multilingual coverage.
less tokens -> less work

Now the prompt is integers but what do they mean?

##### Embeddings

When the tokenizer hands the model an integer, the model looks up that row and uses the vector instead. That vector is that token's embedding. It's the model's representation of what that token "means," learned during training. 

Embedding matrix is a lookup table: Token ID in -> Learned vector out

Semantically similar tokens end up closer. None of this hard-coded, it emerges from training. You can do arithmetic on embeddings and it sometimes work. 
The famous example is king - man + woman = queen.

##### Positional Encoding

How the model gets order info. Plain self-attention doesn't have built-in representation of world order. The original transformer paper did this by giving each position its own pattern of numbers and adding that to the token's embedding before any other processing. The patterns came from sine and cosine waves at different frequencies. 

That worked and the sinusoidal encodings were chosen partly because they can even extrapolate beyond the sequence lengths during training.  However, additive position schemes ran into two problems as we scale up:
- embeddings carry both position and meaning, only so much you can pack in 
- learned absolute position embeddings don't generalize cleanly

Note: 
Additive schemes is the umbrella term for any method that adds position numbers directly.
Sinusoidal encodings are a type of additive position scheme and they still suffer from the first problem (not the second), whereas learned absolute fail both.

RoPE (rotary position embeddings) remains the industry standard as of 2026. 
Instead of adding position info, it rotates Query and Key vectors so relative distance shows up during attention. (position = how far vector is rotated)
A token at position 1 gets a small turn, token at position 100 gets a larger turn. When tokens are later compared during attention, the difference in their Query and Key rotation tells us how far apart they are. 

Why RoPE?
- flawless extrapolation
- no overcrowding
- relative distance not absolute 
- better attention 
- proven performance

LLMs have a "lost in the middle" problem. 
Prompt engineer tip: tell it to "put important context first" / "repeat key info at end"

Meaning & Position! But how do tokens actually exchange info?

##### Attention

Core mathematical technique that lets model weigh the importance of words relative to each other. It does this by giving each token three roles at once. Each token gets transformed into three vectors called Query, Key, and Value.

- Q = what am I looking for 
- K = what I offer to token looking at me
- V = what gets passed along when match happens

Each token's Query is compared against the Key of other token using scaled dot product (intuitively, this measures how aligned the two vectors are)
The match score then gets turned into weights using Softmax. 

Softmax function is a mathematical formula that converts a vector of raw scores into a valid probability distribution where every lies between 0 and 1, and all values sum up to 1.

Example: “The cat that I saw yesterday was sleeping.” When the model processes “was,” it needs to figure out what’s doing the sleeping. The Query vector for “was” gets compared against the Key vectors of the tokens it is allowed to see. The dot product with “cat” is high, because the model has learned that verbs like “was” need a subject and that subjects like “cat” produce Key vectors that line up well. The dot product with “yesterday” is low. Softmax turns those scores into weights, “cat” gets a high weight, “yesterday” gets a low one. The model then takes a weighted sum of the corresponding value vectors, so the value for “cat” dominates the result. The new representation of “was” is now mostly shaped by the value of “cat.” That’s how a token several positions back becomes the referent.

Some definitions: 

- Attention head  is a single independent processing unit within in a neural network's attention layer that computes relationships between words or tokens using its own unique set of learned weight matrices. 

- Induction head is an attention head that notices repeated patterns in the prompt and helps continue them.

- Causal masking is an attention mechanism constraint used in decoder-based AI models that blocks tokens from looking at future positions, ensuring the model only processes past and present inputs.

- In-context learning is a technique where an LLM learns and performs a new task by viewing examples inside a prompt, without changing its internal weights.

- Interpretability research in AI is the scientific study of opening up "black-box" models to understand their internal mechanism.

In full attention, since each token compares against all tokens it's gets pretty expensive the longer the prompt gets. Lot of recent research is about making attention efficient.

##### Multi-head attention

Runs attention many times in parallel because language has many relationships happening at the same time.  

This part that's often described wrong even in tutorials. Each head doesn't get a literal slice of the original token vector but instead has its own projection matrices that map the full token vector down to its own smaller Q, K, and V vectors. 
So if a model has 4,096 numbers per token and 32 heads, each head usually works in a 128-dimensional space, but those 128 numbers are a learned projection of the full 4,096, not a fixed slice. Different "views" of the same token, not different chunks of it.

Each head runs its attention pass independently. Then the outputs of all get concatenated and passed through a final linear layer that mixes them back into one full-size vector. 
The model learns that final mixing too.

![[4.png]]

What's interesting is that Specialization amongst the different heads emerges naturally during training. 

A single transformer layer might have 32 heads. A modern frontier model has dozens of layers (so a typical LLM has 1000s of attention heads, each adding its own learned view)

Recent architectural change:
KV cache: Stores old Key and Value vectors during generation and saves the model from recomputing the whole prompt every time it adds a token. It is the main memory cost of running an LLM at long context lengths.

Grouped-Query Attention lets multiple query heads share fewer key/value heads. That cuts KV-cache memory while keeping many query views.

##### Feed-forward network

Where attention is about tokens talking to each other, FFN is about each token doing more processing. It runs on every token’s vector independently. 
Most of the parameters in a dense transformer model live in the FFN, not in attention.

- expand a token's vector size to a larger size
- apply a nonlinear function
- compress the vector back down to its original size 

![[5.png]]

A non-linearity is a function that bends its input.
It prevents the network from collapsing into one big linear transformation.
The simplest one, ReLU, outputs 0 for -ve number and +ve numbers stay unchanged.

Without it, the FFN would just be two linear layers stacked together. 
Two linear layers in a row are mathematically equivalent to a single linear layer.
Hundred linear layers in a row are still equivalent to one. 
The non-linearity is what stops that collapse, and it’s the reason the FFN can do something richer than a single matrix multiplication.

Researchers have found that some neurons inside the FFN are strongly associated with specific concepts or facts. 
This stored-memory property has an interesting consequence. 
We can edit some facts in a trained model without retraining it. Methods like ROME (Rank-One Model Editing) can change “the Eiffel Tower is in Paris” to “the Eiffel Tower is in Rome” by making a targeted low-rank edit to a specific FFN weight matrix.

Some modern frontier models have started replacing the dense FFN with something called Mixture of Experts (MoE). 
Instead of one FFN per layer, the model has many parallel FFNs (called experts) and it routes each token through only a few of them.

Mixtral 8x7B has 46.7 billion total parameters but uses about 12.9 billion per token.
This has become a common option for very large models because it lets you keep growing the parameter count without making inference cost grow in proportion.

##### Residual stream and layer normalization

Residual connections weren’t invented for transformers. They came from ResNet, originally for image recognition. The motivation was that deep networks were impossible to train. 
The training signal got too weak (or too strong) by the time it traveled back through many layers ( which means the model couldn’t actually learn from its own mistakes)

Adding a shortcut path let the signal flow directly back from the output to the input. Suddenly you could train networks with 100s of layers. Transformers inherited the same trick.

A residual connection adds a block’s output back to the vector it started from. It gives information and gradients a shortcut through the network.

![[6.png]]

Layer normalization rescales a token vector so its numbers stay in a stable range while the model trains.

- The original 2017 transformer applied normalization AFTER each sub-block (post-norm). 
- Modern transformers commonly apply normalization BEFORE each sub-block (pre-norm).

The original layer normalization did two things at once: shift each vector toward zero, then rescale the size of the numbers. RMSNorm drops the shift step and keeps only the rescaling. Empirically, the rescaling carries most of the benefit while being cheaper to compute.

RMSNorm is a cheaper normalization method that rescales vector size without subtracting the mean first.

```
the unglamorous machinery:
without residual connections = deep models become hard to train
without layer normalization = running sum can blow up or collapse
```

##### Next token prediction

After all the layers of attention and FFN finish, the model has a vector for each token in the sequence. During generation, To predict the next word, it takes the final vector of the last token only.

That last vector gets converted into one number per possible next token. 
If the vocabulary has 100,000 tokens, that’s 100,000 numbers. 
These numbers are called logits (which are raw scores for each possible next token, they become probabilities only after softmax which is the same operation as before but different place in the model)

The model usually does not just pick the highest-probability token every time. 
- Decoding settings control how deterministic or varied the output is. 
- Temperature changes how sharp the distribution is (temperature controls randomness)
- Top-k and top-p limit the choices to the most plausible next tokens. That is why the same model can feel precise in one setting and more creative in another.

Once a token is picked, it gets added to the input. 
The model runs the next step on the longer sequence, usually reusing the KV cache.

New attention for the new token. New feed-forward. New final vector. New prediction.

The loop continues until the model emits an end-of-sequence token or hits a length limit. 

This single objective, predicting the next token, is the core training signal for a base LLM. 
The base model isn’t trained on factual accuracy, conversational ability, reasoning, or coding directly. It’s trained to predict the next token in massive amounts of text. Later post-training can then tune the model for instruction following, preference, safety, and conversational behavior.

There’s been a major efficiency innovation worth knowing about. It’s called speculative decoding. A small fast model proposes several tokens ahead. The big model verifies them in parallel. If the proposed tokens are accepted under the big model’s probabilities, accept them. If not, fall back to the big model. Done correctly, the output distribution matches running the big model alone, but the loop can run much faster.

> **Tiny explainer: speculative decoding**  
> Speculative decoding uses a small draft model to guess ahead, then asks the larger model to verify several guessed tokens at once.

The next-token prediction loop is the simplest part of the architecture, but it’s what makes the whole thing work.




























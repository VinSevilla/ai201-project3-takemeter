Community: What community did you choose and why? Why is this community a good fit for a classification task — what makes the discourse varied enough to be interesting?

> The community I chose was Magic the Gathering. I chose this community because I very recently began getting into Magic and the game is very popular with a huge online presence. This card game is also very complex and has a lot of rules which opens the opportunity for a lot of discussion and questions about the rules. The game also has a lot of memes and jokes that are shared within the community which would be interesting to classify as well. Overall, the discourse within the Magic the Gathering community is varied enough to be interesting for a classification task because there are many different types of posts and comments that can be classified into different categories.

Labels: What are your 2–4 labels? Define each in a complete sentence. Include 2 example posts per label.

> The labels associated with this community are memes, rule questions/ clarification, and discussion. Rule questions/clarification posts are those that ask for clarification on the rules of the game or seek answers to specific rule-related questions. Memes are posts that are intended to be humorous and often involve jokes, images, captions, or references related to Magic the Gathering.
> Discussion posts are those that involve more in-depth conversations about various aspects of the game, such as strategy, deck building, or general opinions about the game.

Hard edge cases: What type of post will be genuinely ambiguous between two labels? How will you handle it when you encounter it during annotation?

> In regards to hard edge cases, the identity issue wouldn't primarily be due to ambiguity. These three labels are distinct enough to where their identities can create a complicated mix. I feel the main problem is if a post just incorporated multiple labels simultaneously. For example, a post that is a meme but also includes a question about the rules. In this case, I would handle it by looking at the primary intent of the post. If the main focus is on the meme aspect, I would classify it as a meme. If the main focus is on the question about the rules, I would classify it as a rule question/clarification. If it is truly balanced and cannot be classified as one or the other, I would create a new label for mixed content or simply classify with discussion if it doesn't fit neatly into either category.

Data collection plan: Where will you collect examples? How many per label? What will you do if a label is underrepresented after 200 examples?

> I will collect examples from the Magic the Gathering subreddit, as it is a very active community with a lot of posts and comments. I plan to collect total 200 examples for my data, each label having approximately 67 examples. If a label is underrepresented after collecting 200 examples, I will look for additional sources such as other forums or social media groups dedicated to Magic the Gathering to find more examples for that label.

Evaluation metrics: Which metrics will you use to evaluate your model and why are those the right ones for this specific task? (Accuracy alone is not enough — explain what else you need and why.)

> For evaluating the model aside from accuracy, I will also use diversity metrics such as precision, recall, and F1 score. Precision will help measure how many of the posts classified as a certain label are actually relevant to that label, while recall will measure how many relevant posts were correctly identified by the model. The F1 score will provide a balance between precision and recall, giving a single metric that considers both false positives and false negatives. These metrics are important for this classification task because they will help ensure that the model is not only accurate but also effective in correctly identifying the different types of posts within the Magic the Gathering community.

Definition of success: What performance would make this classifier genuinely useful? What would you accept as "good enough" for deployment in a real community tool?

> A performance that would make this classifier genuinely useful would be achieving an F1 score of at least 0.8 for each label, indicating a good balance between precision and recall. This level of performance would suggest that the model is effectively identifying and classifying posts into the correct categories, making it a valuable tool for users seeking specific types of content within the Magic the Gathering community.

<------------------------------------------------------------------------------------------------------>
AI Tool Plan

> **1. Label stress-testing via boundary-case generation**
> I will prompt an AI tool to generate 5–10 synthetic posts that sit at the boundary between two labels, then use those posts to pressure-test my label definitions before annotation begins. The three boundary pairs to probe are:
>
> - **Rules ↔ Discussion**: Posts that ask a rules question but frame it as a strategic debate (e.g., "Does the timing on this combo actually work, and if so is it even worth running?"). The line: if the primary goal is resolving a specific rule ambiguity.
> - **Memes ↔ Rules**: Posts that wrap a genuine rules question inside a joke format (e.g., a captioned image mocking a confusing interaction while also asking how it resolves). The line: if a reader who already knows the rule would still find the post useful/funny as a meme, classify as Meme; if removing the humor leaves a rules question that still needs answering, classify as Rules.

> - **Memes ↔ Discussion**: Posts that use humor as a framing device for a real opinion or strategy take (e.g., a "starter pack" meme that doubles as deck-building commentary). The line: if the informational content could stand alone as a discussion post, classify as Discussion; if the joke is the whole point, classify as Meme.

2 Label Boundary Case Generation:

> 1. Rules/ Discussion: "Okay so if my opponent uses Doom Blade on my creature in response to my equip activation, does the equip still resolve? And is it even worth equipping mid-combat if this can just happen?"

> 2. Memes/ Rules: [Image of a confused cat staring at a stack of cards] "Me trying to figure out if my opponent can counter my counterspell targeting their counterspell. Can they??"

> 3. Memes/ Discussion: "POV: you finally built a 'budget' deck [image of a $300 pile]. At what point do we admit budget is a myth in this game?"

> 4. Rules/ Discussion: "I know deathtouch + trample means you only need to assign 1 damage to blockers — but does that actually make trample good in EDH or is it just a trap keyword?"

> 5. Memes/ Rules: Lands don't tap for mana, they produce mana, and you use mana to cast spells — distinction matters only when [lists 4 obscure corner cases involving Boseiju]. I made this meme to cope."

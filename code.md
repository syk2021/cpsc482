# Competition-Level Code Generation with AlphaCode, Li et al. (2022)
Sophia Kang<br/>
May 4, 2023

## Paper Walkthrough
### Motivation
    Creating systems that can generate code given problem descriptions requires a deep understanding of both problem 
    solving and reasoning. Recently, large-scale transformer-based language models used to generate texts were shown
    to be capable of generating solutions to simple programming problems in Python. These tasks often involved shorter
    problems that were solvable with shorter solutions that translated a sequence of steps into code. In comparison,
    generating solutions for competitive programming problems is more complex. It involves developing an entire 
    approach from scratch, with strategic use of a multitude of algorithms and data structures. It also often requires
    understanding more complicated problem descriptions. Besides this, feedback attainable is minimal because there are
    private test cases and competitive programming problems become more difficult over time as programmers can rely on
    their solutions for past problems to solve new ones. Because competitive programming is more challenging in this way,
    Li et al. emphasize that developing a machine learning system that can generate code for these problems is a "robust
    and meaningful benchmark for many aspects of intelligence."

### Methodology
    A new code generation system for competitive programming problems called AlphaCode was proposed in 2022 by Li et al. 
    Because creating solutions to competitive programming problems is like a sequence-to-sequence prediction task of 
    evaluating the probability that a generated solution is correct given the problem description in natural language, 
    the authors chose to implement an encoder-decoder transformer model. This model took in the problem description 
    and 8,000 tokens trained on a mix of GitHub and CodeContests data as input and autoregressively sampled generated 
    solutions from the decoder one token at a time until an end-of-code token was produced.

    The main dataset used in this study, CodeContests, included competitive programming problems, correct and incorrect 
    human submissions, and test cases. The training set contained 13,328 problems each with around 922.4 human 
    submissions, while the validation and test sets contained 117 and 165 problems respectively. Each problem in this
    dataset also had metadata such as problem difficulty ratings in the range [800, 3500] with higher ratings indicating
    more difficult problems, problem tags indicating what kinds of algorithms may be useful such as "divide and conquer," 
    "dynamic programming," or "data structures," and programming languages. By nature of competitive programming problems,
    the authors did not have access to all of the hidden test cases for each problem, so they generated extra tests by 
    mutating existing ones. To equate the situation that the model faced with that faced by human progammers, they made
    sure that problems in the training set were ones that had appeared online before all problems in the validation
    and test sets. For comparison, they trained models with varying numbers of parameters, which ranged from 300 million
    (300M model) to 41 billion (41B model).

    The authors pretrained the models on the GitHub dataset with a cross-entropy next-token prediction loss for the
    decoder and a masked language modeling loss for the encoder. They then split GitHub files by uniformly sampling
    pivot locations and using files before the pivot as input to the encoder and files after the pivot as input to the
    decoder. Then the model was fine-tuned on the CodeContests dataset using the problem's natural language description
    for the encoder and the problem's solution for the decoder. In this step, they used techinques such as tempering,
    value conditioning and prediction, and a reinforcement learning algorithm called generation by off-policy learning
    from demonstrations (GOLD) to improve the solve rate. By using tempering, a regularization technique that makes the
    token probability distribution sharper in training, they were able to avoid overfitting the model to the fine-tuning
    dataset. Value conditioning and prediction used both correct and incorrect human submissions to expand the training
    set. Through GOLD, Li et al. were able to focus training on the most likely human solutions out of many other 
    submissions that were accepted as solutions in the CodeContests dataset.

    The study then sampled from the transformer model with a fixed sampling temperature and metadata conditioning. 
    To reflect penalties in competitive programming contests, the authors chose to select ten best samples, or ten best
    submissions per problem, to evaluate with the hidden tests. To do so, they relied on filtering and clustering these 
    samples. Filtering ran the code in sample solutions against example tests given in the problem statement and removed
    samples that failed those tests. While this removed around 99% of generated solutions, the authors still needed to 
    choose ten solutions from the remaining thousands of solutions, at which point they used clustering. The remaining
    solutions were run with inputs that were generated by a separate input generation transformer model and were clustered
    together based on similarity of outputs. For submission, they then picked one sample from each of the ten largest
    clusters.

### Results
    To evaluate how competitive these ten solutions were compared to human-generated code, the authors submitted their
    code to ten Codeforces competitions held from December 1st, 2021 to December 28th, 2021 that each had more than
    5,000 participants. Li et al. found that the system overall achieved an average ranking of top 54.3% when limited to
    ten submissions per problem. They, however, also noted that for 66% of solved problems, 1 submission was enough to 
    solve the problem. The authors then evaluated their models on the validation and test sets using a metric called 10@k.
    This measured the percentage of problems solved when k samples from the model were generated and 10 of them were 
    eventually submitted for evaluation on hidden tests. With up to 100,000 samples per problem (10@100K), the AlphaCode
    41B model solved 29.6% of problems in the CodeContests test set.

    There were four major observations when different models were compared. First, the solve rate scaled approximately
    log-linearly with the number of samples. However, sample sizes could be increased only to a certain level as sample
    sizes needed to be increased exponentially to improve the solve rate. Increasing the training compute, or number of
    parameters, of each model seemed to be an alternative, since solve rate also scaled log-linearly with the number of
    training parameters when near-optimal model sizes were chosen for each training compute size. Third, optimal model
    sizes increased as more training parameters were added to the model. Lastly, out of the different sample selection
    methods, random selection (10@k with no filtering) was the worst performing model and 10@k with filtering and
    clustering, as used in AlphaCode, was the best performing model after perfect sample selection. If filtering was not
    applied, increasing the sample size did not bring improvement in the solve rate. While there still was a noticeable 
    gap with the performance of the perfect sample selection model, 10@k with filtering and clustering proved to be 
    better performing than models without filtering and clustering.

## Thoughts and Comments
    What interests me most about this work is that it demonstrated that machine learning systems have the potential to
    produce complex, useful, and reliable code in the future. Although AlphaCode's performance is only comparable to
    that of a novice programmer now, it suggests that artificial intelligence can produce code that can go beyond short
    solutions to simple programming problems and can be comparable to human solutions for more complicated tasks such as
    competitive programming problems. It was also significant in the sense that it showed improved performance of
    transformer models in code generation, since AlphaCode had a 29.6% solve rate compared to the one-digit solve rate
    that previous studies found. Moreover, while a common concern for language models trained on large amounts of data
    is that they attempt to solve problems by memorizing the entire training set, there was no evidence of AlphaCode 
    doing so, since memorizing was not enough to solve problems in the validation set. When considered alongside the
    recent capabilities of ChatGPT generating code in many different programming languages and finding errors in user
    inputted code, this study shows that machine learning systems may produce more competent code in the future.

    That is not to suggest that this study goes without limitations. Li et al. found that AlphaCode solved more easier 
    problems than more difficult ones. For instance, the 41B model solved 62.4% of validation problems with problem
    difficulty ratings between 800 and 1100, while it solved 7.8% of problems with ratings between 2400 and 2700. The
    model also showed a difference in what types of problems it could solve. Models used in this study had the highest
    solve rates for problems with approaches such as bitmasks, sorting, greedy algorithms, and math but had the lowest
    solve rates for problems that required more complex approaches such as dynamic programming, constructive algorithms,
    and graph problems. This was related to the length of the solutions. Solutions for problems with bitmasks, sorting,
    greedy algorithms, and math were shorter while those for problems with dynamic programming, constructive algorithms,
    and graph problems were longer.

    AlphaCode was also sensitive to problem descriptions that were given as input. When a problem description that had
    asked to find the maximum product of two consecutive array elements was modified to find the minimum product or to
    find the maximum product of any elements instead of consecutive ones, the correctness decreased from 17.1% to 0.1%
    and 3.2% respectively. This indicates that AlphaCode cannot yet apply solutions of one problem to another related 
    problem. Alterations that did not fundamentally change the problem statement, such as typos, affected solve rates less.

    To overcome these shortcomings, having models consider test cases may be useful. Test cases in this model are used to
    evaluate the solutions, but in the case that a problem description is changed, test cases can help the system develop
    more correct outputs. Studies such as Zhang et al. (2023) suggest that applying a planning algorithm in the 
    transformer generation process may be helpful, as ideal code generation "[would] stop early in the generation process"
    when it realizes that output would be wrong and "[would] bias the generation process towards generating successful
    programs" that would ultimately pass more test cases. In addition, improved natural language processing techniques
    may be helpful. As pointed out by Hendler (2023), human programmers use mnemonic variable names and include comments
    that explain their code. Just as how these variable names and comments help human programmers interpret code written
    by others, if machine learning systems develop the ability to understand these, it can help their reasoning process
    when generating code. Besides this, code generated by machine learning systems would also need to be understandable 
    by humans, in case humans need to use the code outputted by these systems. While there are still steps to take to 
    ensure that AI generated code can be used reliably by humans, AI and human programmers may soon be able to work 
    together to produce more accurate and efficient code.

### References
- Hendler, James. "Understanding the Limits of AI Coding." *Science*, vol. 379, no. 6632, 9 Feb. 2023, pp. 548-548.,
  https://doi.org/10.1126/science.adg4246.
- Li, Yujia, et al. "Competition-Level Code Generation with AlphaCode." *Science*, vol. 378, no. 6624, 8 Dec. 2022, 
  pp. 1092-1097., https://doi.org/10.1126/science.abq1158.
- Zhang, Shun, et al. "Planning with Large Language Models for Code Generation." *International Conference on 
  Learning Representations 2023*, 9 Mar. 2023, pp. 1-28., https://doi.org/10.48550/arXiv.2303.05510.


Verifying QA task decomposition
================
PT
2024-12-31

# Explaining answers

The live experiment can be found
[here](https://polina-tsvilodub.github.io/goal-inferences/experiments/qa_explanations/).
Below we look at descriptive stats over different types of explanations
participants generate, when exposed to competitor responses from case
studies 2 and 3. The situations and responses are presented in third
person. These results are from pilots in contexts where the participants
explain the answers to an alien Bo.

``` r
cs2 <- read_csv("../data/qa_explanations/results_8_cs2_pilot2.csv")
cs3 <- read_csv("../data/qa_explanations/results_9_cs3_pilot1.csv")
```

Below we plot the counts of the different explanations. The explanations
were manually classified into the following categories:

- inference: any explanation that made some kind of reference to the
  questioner. For cs2, this often included explanations saying that the
  response offers something that is similar to what the questioner asked
  for. For cs3, it includes both answers generally mentioning that the
  questioner wants the item, and those explaining functional properties
  of the items.
- literal: explanations that focused on the “no” part, saying that the
  item asked for was not available.
- other: unclassifiable explanations.
- similarity: explanations only mentioning the similarity between the
  target and offered option, but not mentioning the questioner.

``` r
cs2_main <- cs2 %>% filter(correct_response == "main") %>%
  mutate(experiment = "case study 2")
cs3_main <- cs3 %>% filter(correct_response == "main") %>%
 mutate(experiment = "case study 3")

explanations_pilot <- rbind(cs2_main, cs3_main)

explanations_pilot %>%
  group_by(experiment, category) %>%
  summarise(
    category_count = n()
  ) %>%
  ggplot(., aes(x = category, y = category_count, fill = category)) +
  geom_col() +
  facet_wrap(experiment~.)
```

    ## `summarise()` has grouped output by 'experiment'. You can override using the
    ## `.groups` argument.

![](pilots_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## Pilot 3

In this pilot for cs 2 materials, we update the instructions on each
main trial to include the following: “…, which thoughts would justify
mentioning the ‘competitor’ instead of mentioning something else”.

Furthermore, we introduce a more fine-grained categorization of
participants’ responses:

- *reference to goals* (clearly stating what the questioner wants to
  do); example:
  - “Netflix is a service that provides movies (along with other
    content). It is possible that the friend may want to watch a movie
    so the statement that they have movies may satisfy their need.”
  - “This response is justified because gingerbread is similar to
    cookies in texture and purpose (a sweet snack), and the flight
    attendant aims to offer something close to what the passenger might
    enjoy.”
  - “Bo, your roommate suggests sunscreen in response to the friend’s
    request for an umbrella since it’s a good strategy to avoid sunburn,
    which is probably the friend’s top concern.”
- *reference to needs* (includes “being helpful to questioner”);
  examples:
  - “The assistant clearly answers the question by saying that they
    don’t have what he is looking for but offers an alternative that is
    more helpful than simply stopping the conversation there (which
    would have been less helpful). The alternative may or may not be
    acceptable but is offered just in case”
  - “He responded with a circus show because he sees the woman is
    looking for a theatre performance for children. They do not have a
    theatre performance, but the circus show is for children if she is
    looking for something different.”
  - “…another option they might like”
  - “The class that was suggested is close enough to what the student
    wanted, the one that was not offered.”
  - “A ziplock bag could be used instead of a box.”
  - “The response is responsible in the sense that she can use the hair
    dryer to dry her hair.”
- *reference to desires / interests / preferences* (of the questioner),
  more likely mentioning some features
  - “Firstly since they don’t have the item the assistant must say they
    do not have it. The assistant however wants to be helpful and knows
    they are interested in Jewellery and therefore offers a plausible
    alternative that they may or may not be interested in”
- *similarity* (or relatedness / being reasonable) of the items without
  mentioning the questioner; examples:
  - “Once again, he is simply explaining that what she is asking they do
    not have, but he is offering an alternative.”
  - “Thrillers are closely related to mysteries as they are both
    exciting and keep the reader guessing about the ending.”
- *feature* (mentioning relevant functional dimension); example:
  - “Neither a cheese pizza nor a pepperoni pizza have vegetables on
    them.”
  - “This is a reasonable response because he doesn’t not own a broom,
    however a vacuum is capable of doing the same function as a broom.”
- *literal*; example:
  - “The cashier responds by saying what they currently sell which is
    mystery novels.”
- *other* unclassifiable responses

``` r
cs2_pilot3 <- read_csv("../data/qa_explanations/results_12_cs2_pilot3.csv")
```

    ## Rows: 55 Columns: 18
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (11): answer, fine_category, comments, correct_response, education, expl...
    ## dbl  (7): submission_id, age, experiment_duration, experiment_end_time, expe...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
cs2_pilot3_main <- cs2_pilot3 %>% filter(correct_response == "main") %>%
  mutate(experiment = "case study 2 pilot 3")

explanations_pilot_fineCat <- rbind(cs2_main %>% select(-category),
                                    cs3_main %>% select(-category), 
                                    cs2_pilot3_main)

explanations_pilot_fineCat %>%
  filter(fine_category != "other") %>%
  group_by(experiment, fine_category) %>%
  summarise(
    category_count = n()
  ) %>%
  ggplot(., aes(x = fine_category, y = category_count, fill = fine_category)) +
  geom_col() +
  facet_wrap(experiment~.) +
  theme(axis.text.x = element_text(angle = 30, hjust = 1))
```

    ## `summarise()` has grouped output by 'experiment'. You can override using the
    ## `.groups` argument.

![](pilots_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

# Inferring goals from questions

The live experiment can be found
[here](https://polina-tsvilodub.github.io/goal-inferences/experiments/goal_sampling/).
Below we preprocess and analyse the results for the pilot where
participants suggested three possible goals a questioner had in mind,
given the context and the question. The context included the three
available options (competitor, same category, other category). Again,
the participants’ task was to explain questioning behavior to an alien
Bo. The results are for vignettes from case study 2 only.

``` r
goals_cs2 <- read_csv("../data/goal_sampling/results_7_cs2_pilot1.csv") %>%
  filter(correct_response == "main")
goals_cs2_long <- goals_cs2 %>%
  pivot_longer(cols = c(answer1, answer2, answer3), names_to = "answer_num", values_to = "sampled_goals")
#goals_cs2_long %>% write_csv("../data/goal_sampling/results_7_cs2_pilot1_long.csv")

goals_cs2_long <- read_csv("../data/goal_sampling/results_7_cs2_pilot1_long.csv")
```

Each of the suggested goals was manually categorized into the following
categories:

- target: includes goals stating that the questioner wants the actual
  target item, or stating the relevant feature of the target (e.g.,
  “wants an iced non-alcoholic beverage”).
- superordinate: includes more general goals, which are mostly epistemic
  (e.g., something like learning what is on the menu).
- subordinate: includes all other more specific goals. Sometimes the
  distinction between super- and subordinate wasn’t 100% clear; some
  subordinate goals are, e.g., “wants to know what is on the menu to
  tell a friend about it”.

Some spill-over between the options and the produced goals was observed;
i.e., some responses assumed that the questioner knows the options,
which should not be the case. These were mostly put towards subordinate
goals. The next pilot removes the options from context.

``` r
goals_cs2_long %>%
  group_by(category) %>%
  summarize(category_prop = n() / nrow(goals_cs2_long)) %>%
  ggplot(., aes(x = category, y = category_prop, fill = category)) +
  geom_col()
```

![](pilots_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Pilot 2

In this pilot, we remove the available options from the context. The
participants only see the contextual setting and the question.

``` r
goals_cs2_p2 <- read_csv("../data/goal_sampling/results_10_cs2_pilot2.csv") %>%
  filter(correct_response == "main")
goals_cs2_p2_long <- goals_cs2_p2 %>%
  pivot_longer(cols = c(answer1, answer2, answer3), names_to = "answer_num", values_to = "sampled_goals")
#goals_cs2_p2_long %>% write_csv("../data/goal_sampling/results_10_cs2_pilot2_long.csv")

goals_cs3 <- read_csv("../data/goal_sampling/results_11_cs3_pilot1.csv") %>%
  filter(correct_response == "main")
goals_cs3_long <- goals_cs3 %>%
  pivot_longer(cols = c(answer1, answer2, answer3), names_to = "answer_num", values_to = "sampled_goals")
#goals_cs3_long %>% write_csv("../data/goal_sampling/results_11_cs3_pilot1_long.csv")

goals_cs2_p2_long <- read_csv("../data/goal_sampling/results_10_cs2_pilot2_long.csv") %>%
  mutate(experiment = "case study 2")
goals_cs3_long <- read_csv("../data/goal_sampling/results_11_cs3_pilot1_long.csv") %>%
   mutate(experiment = "case study 3") %>% filter(category != "other")
goals_long <- rbind(goals_cs2_p2_long, goals_cs3_long)
```

The goals were manually annotated into the same three categories as in
pilot 1, but included a more fine-grained distinction between “general”
goals and “feature”-mentioning goals. The feature-mentioning goals
appealed to the relevant feature of the target (e.g., “wants to do
martial arts” rather than just saying “wants to do kickboxing”).
Generally, compared to the pilot above, the variation in the contents
was larger, the subordinate responses more often mentioned social goals
(e.g., the question as an excuse to start a conversation with the
cashier). In case study 3, there were a few particular aspects to be
noted: the larger proportion of feature-target responses is partially
driven by repetitions of features from the context (e.g., “wants to
reach the window”). The distinction between feature- and non-feature
subordinate / superordinate goals was a bit more difficult to draw
(e.g., it was unclear whether goals like “to avoid cutting or slippery
when holding the mirror” were referencing a feature in the same sense as
the target feature of a blanket being soft).

``` r
goals_long_processed <- goals_long %>%
  mutate(category_split = str_split(category, "-")) %>% 
  unnest_wider(category_split, names_sep = "wide") %>%
  rename(level = category_splitwide1, is_feature = category_splitwide2) %>%
  mutate(is_feature = ifelse(is.na(is_feature), "general", "feature"))

goals_long_processed %>%
  group_by(experiment) %>%
  mutate(expt_count = n()) %>%
  group_by(experiment, level, is_feature) %>%
  summarize(category_prop = n() / expt_count) %>%
  unique() %>%
  ggplot(., aes(x = level, y = category_prop, fill = is_feature)) +
  geom_col() +
  facet_wrap(experiment~.)
```

    ## Warning: Returning more (or less) than 1 row per `summarise()` group was deprecated in
    ## dplyr 1.1.0.
    ## ℹ Please use `reframe()` instead.
    ## ℹ When switching from `summarise()` to `reframe()`, remember that `reframe()`
    ##   always returns an ungrouped data frame and adjust accordingly.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## `summarise()` has grouped output by 'experiment', 'level', 'is_feature'. You
    ## can override using the `.groups` argument.

![](pilots_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

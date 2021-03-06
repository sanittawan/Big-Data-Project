Visualizations
================
Adam Shelton

## MPI Processing

``` r
mpi_data = read_csv(here("output_data", "mpi_trials.csv"))

# create line plot with 1/x smoothing
mpi_data %>% subset(hosts == 1) %>% ggplot(aes(x = nodes, y = proc_time/60)) + 
    geom_smooth(color = color_pal(1, "cool"), size = 1.75, method = "lm", 
        formula = y ~ I(1/x)) + scale_x_continuous(breaks = c(1, 
    2, 4, 8, 16)) + labs(title = "Parallelization has Diminishing Returns", 
    x = "MPI Nodes", y = "Average Running Time (min)") + theme_master(base_size = 22) + 
    theme(panel.grid.minor.x = element_blank())
```

![](visualizations_files/figure-gfm/mpi-1-1.svg)<!-- -->

``` r
# calculate price for each MPI test, converting time to hours
# and requiring two CPUs per node
mpi_data$total_price = 2 * mpi_data$nodes * (mpi_data$proc_time/3600) * 
    0.031611

# create line plot
mpi_data %>% subset(hosts == 1) %>% ggplot(aes(x = nodes, y = total_price)) + 
    geom_smooth(color = color_pal(1, "discrete"), size = 1.75, 
        method = "loess", span = 0.5) + scale_x_continuous(breaks = c(1, 
    2, 4, 8, 16)) + scale_y_continuous(limits = c(0, 0.02)) + 
    labs(title = "Parallelization is More Costly", x = "MPI Nodes", 
        y = "Mean Total Processing Cost (USD)") + theme_master(base_size = 22) + 
    theme(panel.grid.minor.x = element_blank())
```

![](visualizations_files/figure-gfm/mpi-cost-1.svg)<!-- -->

``` r
# get means and standard deviations for each number of hosts
mpi_means = mpi_data %>% filter(nodes == 16) %>% filter(hosts == 
    1) %>% .$proc_time %>% tibble(hosts = c(1), proc_time = mean(.), 
    pc_sd = sd(.)) %>% .[1, 2:4]
mpi_means[2, ] = mpi_data %>% filter(nodes == 16) %>% filter(hosts == 
    2) %>% .$proc_time %>% tibble(proc_time = mean(.), pc_sd = sd(.)) %>% 
    .[1, 2:3] %>% bind_cols(host = c(2), .)
mpi_means[3, ] = mpi_data %>% filter(nodes == 16) %>% filter(hosts == 
    4) %>% .$proc_time %>% tibble(proc_time = mean(.), pc_sd = sd(.)) %>% 
    .[1, 2:3] %>% bind_cols(host = c(4), .)

# plot a column plot with error bars
mpi_means %>% ggplot(aes(x = factor(hosts), y = proc_time)) + 
    geom_col(aes(fill = factor(hosts))) + geom_errorbar(aes(ymin = proc_time - 
    pc_sd, ymax = proc_time + pc_sd), width = 0.2, position = position_dodge(0.9)) + 
    scale_fill_manual(values = color_pal(3)) + labs(title = "Number of MPI Hosts Affects Performance", 
    x = "MPI Hosts", y = "Average Running Time (s)") + theme_master(base_size = 22) + 
    hide_x_gridlines + hide_legend
```

![](visualizations_files/figure-gfm/mpi-2-1.svg)<!-- -->

## Top Tags

``` r
# arrange tags by the number of posts for each one
tt_data = read_csv(here("output_data", "top_tags.csv"))
tt_data$count = as.numeric(tt_data$count)
tt_data = tt_data %>% arrange(-count)
tt_data$language = tt_data$language %>% factor()

# take the top six tags and column plot them
tt_data %>% .[1:6, ] %>% ggplot(aes(x = reorder(language, -count), 
    y = (count/10^6))) + geom_col(aes(fill = language)) + scale_fill_manual(values = color_pal(6)) + 
    labs(title = "Top Six Tags in StackOverflow Posts", x = "Tag", 
        y = "Number of Posts (in millions)") + theme_master(base_size = 22) + 
    hide_x_gridlines + hide_legend
```

![](visualizations_files/figure-gfm/top-tags-1.svg)<!-- -->

## Questions by Year

``` r
# remove quotation marks and whitespace from each question
ques_year_data = read_csv(here("output_data", "top_questions.csv"))
ques_year_data$question = ques_year_data$question %>% str_remove_all("\"") %>% 
    str_remove_all("\\\\") %>% unquote(deep = TRUE) %>% str_trim()
ques_year_data = ques_year_data %>% filter(year != 2012)  # remove weird results for 2012

# plot flipped axis column plot with questions as labels on
# the plot
ques_year_data %>% ggplot(aes(x = factor(year), y = count)) + 
    geom_col(aes(fill = year)) + scale_fill_gradientn(colors = color_pal(2)) + 
    geom_label(aes(x = 1:length(year), y = 110, label = question), 
        alpha = 0.9, color = "black", family = "Pragati Narrow", 
        size = 6, hjust = 0) + labs(title = "Top Questions on StackOverflow by Year", 
    x = "Year", y = "Number of Answers") + theme_master(base_size = 22) + 
    hide_y_gridlines + hide_legend + coord_flip()
```

![](visualizations_files/figure-gfm/ques-year-1.svg)<!-- -->

## Two-Grams

``` r
two_grams_data = read_csv(here("output_data", "twograms.csv")) %>% 
    arrange(-count)

# make graph from top 150 tags
bigram_graph <- two_grams_data[1:150, ] %>% graph_from_data_frame()

remove_axes <- theme(axis.text = element_blank(), axis.line = element_blank(), 
    axis.ticks = element_blank(), panel.border = element_blank(), 
    panel.grid = element_blank(), axis.title = element_blank())

# plot a network graph of these top 150 tags
ggraph(bigram_graph, layout = "fr") + geom_edge_link(color = "grey") + 
    geom_node_point(color = "grey") + geom_node_text(aes(label = name, 
    color = name %in% tt_data$language[1:6], size = name %in% 
        tt_data$language[1:6]), nudge_x = 0, nudge_y = 0, repel = TRUE, 
    family = "Pragati Narrow") + scale_color_manual(values = c("#000000", 
    color_pal(1, "discrete"))) + scale_size_manual(values = c(5, 
    8)) + labs(title = "Connections Between Tags in StackOverflow Posts") + 
    theme_master(base_size = 22) + remove_axes + hide_legend
```

![](visualizations_files/figure-gfm/two-grams-1.svg)<!-- -->

## User Activities

``` r
user_act_data = read_csv(here("output_data", "user_ac_out.csv"))
user_act_data = user_act_data[is.numeric(user_act_data$user_id), 
    ]

# density plot of user activity
ggplot(user_act_data, aes(x = count)) + geom_density(color = color_pal(1, 
    "cool"), size = 1.75) + labs(title = "Most Users Have Very Little Account Activity", 
    x = "Account Interactions", y = "Density") + theme_master(base_size = 22)
```

![](visualizations_files/figure-gfm/user-act-1.svg)<!-- -->

``` r
aggr_ua_data = user_act_data$count %>% table() %>% as.tibble()
names(aggr_ua_data) = c("act_count", "num_obs")
aggr_ua_data$act_count = as.numeric(aggr_ua_data$act_count)
aggr_ua_data$perc = aggr_ua_data$num_obs/sum(aggr_ua_data$num_obs)

# plot treemap of user activity data
ggplot(aggr_ua_data, aes(area = as.numeric(num_obs), fill = as.numeric(act_count), 
    label = act_count)) + geom_treemap() + geom_treemap_text(colour = "white", 
    place = "centre", grow = TRUE) + scale_fill_gradientn(trans = "log10", 
    colors = color_pal(5, type = "continuous")) + labs(title = "The Majority of StackOverflow Accounts Have Very Little Account Activity", 
    fill = "Number of \nPosts") + theme_master()
```

![](visualizations_files/figure-gfm/usr-act-tmap-1.svg)<!-- -->

## User Locations

``` r
user_loc_data = read_csv(here("output_data", "users_gold_badge_locations.csv"))
world_map = map_data("world")

# plot world map with 2d density plot (heatmap) of user
# activity
ggplot() + geom_map(data = world_map, map = world_map, aes(x = long, 
    y = lat, group = group, map_id = region), fill = "white", 
    colour = "#7f7f7f", size = 0.5) + stat_density2d(data = user_loc_data, 
    aes(x = lon, y = lat, fill = stat(level)), geom = "polygon", 
    alpha = 0.75) + scale_fill_gradientn(colors = color_pal(5, 
    "continuous", reverse = TRUE)) + theme_map() + theme(legend.position = "right") + 
    labs(title = "Global Distribution of StackOverflow Users", 
        fill = "Density") + coord_map("albers", 0, 0)
```

![](visualizations_files/figure-gfm/user-loc-1.svg)<!-- --> \`\`\`

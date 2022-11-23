---
title: "Statistics and Graphing with R"
teaching: 10
exercises: 50
questions:
- "How do I use ggplot and other programs to visualize metabarcoding data?"
- "What statistical approaches do I use to explore my metabarcoding results?"
objectives:
- "Create bar plots of taxonomy and use them to show differences across the metadata"
- "Create distance ordinations and use them to make PCoA plots"
- "Compute species richness differences between sampling locations"
keypoints:
- "A basic tutorial to conduct statistical analyses of your data"
---

# Statistical analysis in R

In this last section, we'll go over a bried introduction on how to analyse the data files you've created during the bioinformatic analysis, i.e., the frequency table and the taxonomy assignments. Today, we'll be using **R** for the statistical analysis, though other options, such as **Python**, could be used as well. Both of these two coding languages have their benefits and drawbacks, but because most people know **R** from their Undergraduate classes, we will stick with this option for now.

Please keep in mind that we will only be giving a brief introduction to the statistical analysis. This is by no means a full workshop into how to analyse metabarcoding data. Furthermore, we'll be sticking to basic R packages to keep things simple and will not go into dedicated metabarcoding packages, such as  <a href="https://joey711.github.io/phyloseq/" target="_blank" rel="noopener noreferrer"><b>Phyloseq</b></a>.

## Importing data in R

Before we can analyse our data, we need to import and format the files we created during the bioinformatic analysis, as well as the metadata file. We will also need to load in the necessary R libraries.

```R
###################
## LOAD PACKAGES ##
###################
library('vegan')
library('lattice')
library('ggplot2')
library('labdsv', lib.loc = "/nesi/project/nesi02659/obss_2022/R_lib") # if not on nesi remove the lib.loc...)
library('tidyr')
library('tibble')
library('reshape2')
library('viridis')

#################
## FORMAT DATA ##
#################
## read in the data and metadata
df <- read.delim('../final/otutable.txt', check.names = FALSE, row.names = 1)
tax <- read.delim('../final/taxonomy_otus.txt', check.names = FALSE, row.names = 1, header = FALSE)
meta <- read.csv('metadata.csv', check.names = FALSE)

## merge taxonomy information with OTU table, drop rows without taxID, and sum identical taxIDs
merged <- merge(df, tax, by = 'row.names', all = TRUE)
merged <- subset(merged, select=-c(Row.names,V2, V3))
merged[merged == ''] = NA
merged <- merged %>% drop_na()
merged <- aggregate(. ~ V4, data = merged, FUN = sum)

## remove singleton detections, followed by columns that sum to zero and set taxID as row names
merged[merged == 1] <- 0
merged <- merged[, colSums(merged != 0) > 0]
rownames(merged) <- merged$V4
merged$V4 <- NULL

## transform dataset to presence-absence
merged_pa <- (merged > 0) * 1L 
```

## Preliminary data exploration

One of the first things we would want to look at, is to determine if sufficient sequencing depth was achieved. We can do this by creating rarefaction curves, which work similar to tradtional species accumulation curves, whereby we randomly select sequences from a sample and determine if new species or OTUs are being detected. If the curve flattens out, it gives the indication sufficient sequencing has been conducted to recover most of the diversity in the dataset.

```R
########################
## RAREFACTION CURVES ##
########################
## transpose the original dataframe and remove negative control
df_t <- t(df)
neg_remove <- c('ASN')
df_t <- df_t[!(row.names(df_t) %in% neg_remove),]

## identify the lowers number of reads for samples and generate rarefaction curves
raremax_df <- min(rowSums(df_t))
rarecurve(df_t, step = 100, sample = raremax_df, col = 'blue', cex = 0.6)
```

![png](../fig/rarefaction_curve.png)

The next step of preliminary data exploration, is to determine what is contained in our data. We can do this by plotting the abundance of each taxon for every sample in a stacked bar plot.

```R
######################
## DATA EXPLORATION ##
######################
## plot abundance of taxa in stacked bar graph
ab_info <- tibble::rownames_to_column(merged, "taxID")
ab_melt <- melt(ab_info)
ggplot(ab_melt, aes(fill = taxID, y = value, x = variable)) +
  geom_bar(position = 'stack', stat = 'identity')
```

![png](../fig/sample_abundance.png)

Since the number of sequences differ between samples, it might sometimes be easier to use relative abundances to visualise the data.

```R
## plot relative abundance of taxa in stacked bar graph
ggplot(ab_melt, aes(fill = taxID, y = value, x = variable)) + 
  geom_bar(position="fill", stat="identity")
```

![png](../fig/sample_relative_abundance.png)

While we can start to see that there are some differences between sampling locations with regards to the observed fish, let's combine all samples within a sampling location to simplify the graph even further. We can do this by summing or averaging the reads.

```R
# combine samples per sample location and plot abundance and relative abundance of taxa in stacked bar graph
location <- merged
names(location) <- substring(names(location), 1, 2)
locsum <- t(rowsum(t(location), group = colnames(location), na.rm = TRUE))
locsummelt <- melt(locsum)
ggplot(locsummelt, aes(fill = Var1, y = value, x = Var2)) +
  geom_bar(position = 'stack', stat = 'identity')
ggplot(locsummelt, aes(fill = Var1, y = value, x = Var2)) + 
  geom_bar(position="fill", stat="identity")

locav <- sapply(split.default(location, names(location)), rowMeans, na.rm = TRUE)
locavmelt <- melt(locav)
ggplot(locavmelt, aes(fill = Var1, y = value, x = Var2)) +
  geom_bar(position = 'stack', stat = 'identity')
ggplot(locavmelt, aes(fill = Var1, y = value, x = Var2)) + 
  geom_bar(position="fill", stat="identity")
```

![png](../fig/location_abundance.png)

![png](../fig/location_relative_abundance.png)

As you can see, the graphs that are produced are easier to interpret, due to the limited number of stacked bars. However, it might still be difficult to identify which species are occurring in only one sampling location, due to the number of species detected and the number of colors used. A trick to use here, is to plot the data per species for the two locations.

```R
## plot per species for easy visualization
ggplot(locsummelt, aes(fill = Var2, y = value, x = Var2)) +
  geom_bar(position = 'dodge', stat = 'identity') +
  scale_fill_viridis(discrete = T, option = "E") +
  ggtitle("Graphing data per species") +
  facet_wrap(~Var1) +
  theme(legend.position = "non") +
  xlab("")
```

![png](../fig/per_species_abundance.png)

Plotting the relative abundance (transforming it indirectly to presence-absence) could help with distinguishing presence or absence in a sampling location, which could be made difficult due to the potentially big difference in read number.

```R
## plotting per species on relative abundance allows for a quick assessment 
## of species occurrence between locations
ggplot(locsummelt, aes(fill = Var2, y = value, x = Var2)) +
  geom_bar(position = 'fill', stat = 'identity') +
  scale_fill_viridis(discrete = T, option = "E") +
  ggtitle("Graphing data per species") +
  facet_wrap(~Var1) +
  theme(legend.position = "non") +
  xlab("")
```

![png](../fig/per_species_relative_abundance.png)

## Alpha diversity analysis

Since the correlation between species abudance/biomass and eDNA signal strength has not yet been proven, alpha diversity analysis for metabarcoding data is currently restricted to species richness comparison. In the near future, however, we hope the abundance correlation is proven, enabling more extensive alpha diversity analyses.

```R
######################
## SPECIES RICHNESS ##
######################
## calculate number of taxa detected per sample and group per sampling location
specrich <- setNames(nm = c('colname', 'SpeciesRichness'), stack(colSums(merged_pa))[2:1])
specrich <- transform(specrich, group = substr(colname, 1, 2))

## test assumptions of statistical test, first normal distribution, next homoscedasticity
histogram(~ SpeciesRichness | group, data = specrich, layout = c(1,2))
bartlett.test(SpeciesRichness ~ group, data = specrich)

## due to low sample number per location and non-normal distribution, run Welch's t-test and visualize using boxplot
stat.test <- t.test(SpeciesRichness ~ group, data = specrich, var.equal = FALSE, conf.level = 0.95)
stat.test
boxplot(SpeciesRichness ~ group, data = specrich, names = c('AM', 'AS'), ylab = 'Species Richness')
```

![png](../fig/species_richness_comp.png)

## Beta diversity analysis

One final type of analysis we will cover in today's workshop is in how to determine differences in species composition between sampling locations, i.e., beta diversity analysis. Similar to the alpha diversity analysis, we'll be working with a presence-absence transformed dataset.

Before we run the statistical analysis, we can visualise differences in species composition through ordination.

```R
#############################
## BETA DIVERSITY ANALYSIS ##
#############################
## transpose dataframe
merged_pa_t <- t(merged_pa)
dis <- vegdist(merged_pa_t, method = 'jaccard')
groups <- factor(c(rep(1,6), rep(2,5)), labels = c('mudflats', 'shore'))
dis
groups
mod <- betadisper(dis, groups)
mod

## graph ordination plot and eigenvalues
plot(mod, hull = FALSE, ellipse = TRUE)
ordination_mds <- wcmdscale(dis, eig = TRUE)
ordination_eigen <- ordination_mds$eig
ordination_eigenvalue <- ordination_eigen/sum(ordination_eigen)
ordination_eigen_frame <- data.frame(Inertia = ordination_eigenvalue*100, Axes = c(1:10))
eigenplot <- ggplot(data = ordination_eigen_frame, aes(x = factor(Axes), y = Inertia)) +
  geom_bar(stat = "identity") +
  scale_y_continuous(expand = c(0,0), limits = c(0, 50)) +
  theme_classic() +
  xlab("Axes") +
  ylab("Inertia %") +
  theme(axis.ticks.x = element_blank(), axis.text.x = element_blank())
eigenplot
```

![png](../fig/ordination.png)

![png](../fig/eigenvalue.png)

Next, we can run the statistical analysis PERMANOVA to determine if the sampling locations are significantly different. A PERMANOVA analysis will give a significant result when (i) the centroids of the two sampling locations are different and (ii) if the dispersion of samples to the centroid within a sampling location are different to the other sampling location.

```R
source <- adonis(merged_pa_t ~ Location, data = meta, by = 'terms')
print(source$aov.tab)
```

Since PERMANOVA can give significant p-values in both instances, we can run an additional test that only provides a significant p-value if the dispersion of samples to the centroid are different between locations.

```R
## test for dispersion effect
boxplot(mod)
set.seed(25)
permutest(mod)
```

A final test we will cover today, is to determine which taxa are driving the difference in species composition observed during the PERMANOVA and ordination analyses. To accomplish this, we will calculate indicator values for each taxon and identify differences based on `IndValij = Specificityij * Fidelityij * 100`.

```R
## determine signals driving difference between sampling locations
specrich$Group <- c(1,1,1,1,1,1,2,2,2,2,2)
ISA <- indval(merged_pa_t, clustering = as.numeric(specrich$Group))
summary(ISA)
```

**That is all for the OBSS 2022 eDNA workshop. Thank you all for your participation. Please feel free to contact me for any questions (gjeunen@gmail.com)!**

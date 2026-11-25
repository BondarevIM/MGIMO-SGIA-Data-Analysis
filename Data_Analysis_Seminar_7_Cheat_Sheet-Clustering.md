<div style="text-align: center; font-family: 'Times New Roman', Times, serif; font-size: 16pt; font-weight: bold">

# Cheat Sheet for Data Analysis #7<br>Clustering
</div>

<div style="font-family: 'Times New Roman', Times, serif; font-size: 12pt; text-align: left">

## Inspect the data

### Check dimensions and missing values

```python
print(df.shape)
print(df.isna().sum())
```
*Meaning:* Confirms the number of observations and locates missing values before you fit a model.

### Select numerical features

```python
features = df.drop(columns=["name"])
```
*Meaning:* Use the numerical indicators that define similarity. Keep names or IDs in the original table so you can identify observations.

---

## Standardise the features

### Centre and scale each column

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(features)
```
*Meaning:* Subtracts each feature’s mean and divides by its standard deviation. This reduces the influence of differences in measurement units on Euclidean distances. Check that the chosen features are relevant.

---

## Fit k-means

### Choose a number of clusters and fit the model

```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, init="random",
               n_init=10, random_state=0)
model.fit(X_scaled)
```
*Meaning:* Fits three clusters. The settings fix the random starting sequence and use ten starts. The model retains the run with the smallest sum of squared distances to cluster centres.

### Attach and count the labels

```python
df_clusters = df.assign(Cluster=model.labels_.astype(str))
sizes = df_clusters["Cluster"].value_counts().sort_index()
print(sizes)
```
*Meaning:* Adds one label per row to a new table. Text labels tell plotting functions to treat clusters as categories. The numbers identify groups and carry no ranking.

---

## Plot cluster labels

### Colour and mark the groups on a scatter plot

```python
import matplotlib.pyplot as plt
import seaborn as sns

sns.scatterplot(data=df_clusters, x="indicator_a", y="indicator_b",
                hue="Cluster", style="Cluster")
plt.show()
```
*Meaning:* Shows the group assignment on two original axes. Name the axes and their units in the chart. The model can use more variables than the plot shows.

---

## Compare candidate partitions

### Calculate the average silhouette

```python
from sklearn.metrics import silhouette_score

score = silhouette_score(X_scaled, model.labels_)
print(round(score, 3))
```
*Meaning:* Compares distances within a cluster with distances to its nearest alternative. A higher score supports better separation by this measure on the given features. A silhouette needs at least two distinct labels and fewer labels than observations.

### Compare scores and sizes on the same inputs

```python
rows = []
for k in [3, 4, 5]:
    candidate = KMeans(n_clusters=k, init="random",
                       n_init=10, random_state=0)
    candidate.fit(X_scaled)
    counts = pd.Series(candidate.labels_).value_counts().sort_index()
    score = silhouette_score(X_scaled, candidate.labels_)
    rows.append({"k": k, "Silhouette": round(score, 3),
                 "Cluster sizes": counts.tolist()})
print(pd.DataFrame(rows))
```
*Meaning:* Varies only the number of groups. Read each score together with the sizes of its clusters. A small group warrants a check of its members.

---

## Profile the groups

### Report medians in the original units

```python
columns = ["indicator_a", "indicator_b"]
medians = df_clusters.groupby("Cluster")[columns].median()
print(medians.round(3))
```
*Meaning:* Describes each group using the original indicators. Use the same label column when you report sizes and medians.

### Inspect selected observations

```python
selected = df_clusters["name"].isin(["Observation A", "Observation B"])
df_clusters.loc[selected, ["name", "indicator_a", "indicator_b", "Cluster"]]
```
*Meaning:* Checks which labels were assigned to individual rows and what values are visible on the two plotted axes.

---

## Optional comparison with Ward’s method

### Use the same features and group count

```python
from sklearn.cluster import AgglomerativeClustering

ward = AgglomerativeClustering(n_clusters=3, linkage="ward")
ward_labels = ward.fit_predict(X_scaled)
print(pd.crosstab(model.labels_, ward_labels))
```
*Meaning:* Compares membership under two procedures. Read the counts across rows and columns because each model names its groups independently.

---

## Practical Tips

1. Choose the features before you fit. Check their meaning, missing values and scales.
2. Keep the original units for the plot and the description. Use the matrix supplied to the model for the silhouette.
3. Keep the features and initialisation settings fixed when comparing candidate values of k.
4. Inspect group sizes and profiles before choosing a partition. A chart with two axes can hide differences on the other indicators.

Remember: A cluster describes similarity under a stated choice of features, scale and method. Report those choices alongside the result.

## References

- [StandardScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html)
- [KMeans](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html)
- [silhouette_score](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html)
- [seaborn.scatterplot](https://seaborn.pydata.org/generated/seaborn.scatterplot.html)
- [AgglomerativeClustering](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html)

</div>

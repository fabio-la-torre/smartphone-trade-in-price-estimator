# Smartphone Trade-in Price Estimator
Building AI course project - Estimates the trade-in value of a used smartphone

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fabio-la-torre/smartphone-trade-in-price-estimator/blob/main/trade_in_price_estimator.ipynb)

## Summary

Building AI course project. It estimates the trade-in value of a used smartphone from features such as new price, release year, days of use and RAM. It uses linear regression trained on about 3,000 real phones and explains 82% of the price differences (R² = 0.82).


## Background

I work as a sales consultant in an electronics store and handle at least 20 smartphone trade-ins every week. Today the valuation comes from a closed system: it requires the physical phone, its IMEI code and the customer's personal data. It is a long process, and when a customer asks "how much is my phone worth?" without having it with them, there is no way to give an answer.

The problems this project addresses:
* customers have no quick and transparent idea of what their phone is worth
* without the device, and therefore without the IMEI, no estimate is possible
* the valuation is a "black box": nobody knows which features really matter

My motivation is to understand what is inside that black box: how can an algorithm arrive at a price starting only from the features of a phone?

There is also an environmental side: every resold smartphone keeps being used instead of becoming waste, and it is one less phone to produce.


## How is it used?

The tool is designed for the salesperson in the store, in situations where the official valuation system cannot be used: the customer knows which model they own, but does not have the phone with them, and therefore has no IMEI code.

The salesperson enters eight values that can easily be found in the model's technical specifications (new price, release year, days of use, RAM, battery, screen size, cameras) and immediately gets an indicative estimate in euros to share with the customer.

**Important:** the estimate is indicative and does not replace the official valuation. The model is trained on data collected in 2021 and does not take into account the physical condition of the phone (scratches, damage, battery wear). This is a study project.

### How to try it
1. Open the notebook with the **Open in Colab** button above
2. Upload the file `used_device_data.csv` (included in this repository) to the Colab file panel
3. Run all cells (**Runtime → Run all**)

Example:
```python
# 300 € phone, released 2020, used 300 days, 4 GB RAM, 4000 mAh, 6.2", 10 MP + 10 MP
estimate_price(300, 2020, 300, 4, 4000, 6.2, 10, 10)
# -> 124 €
```


## Data sources and AI methods

### Data
The dataset is the [Used Phones & Tablets Pricing Dataset](https://www.kaggle.com/datasets/ahsan81/used-handheld-device-data), published on Kaggle under a CC0 license (public domain). It contains 3,454 devices with data collected in 2021: brand, operating system, screen size, cameras, storage, RAM, battery, weight, release year, days of use, and new and used prices (on a logarithmic scale).

Data preparation:
* tablets removed (screen larger than 7", i.e. 17.8 cm): 275 devices
* rows with missing values removed
* 2,991 smartphones remain, split into 80% for training (2,392) and 20% for testing (599)

### Method
The model uses **linear regression**: the used price is estimated as the sum of the phone's features, each multiplied by a weight.

*estimated price = w₀ + w₁ · new price + w₂ · release year + … + w₈ · front camera*

During training the model finds the weights that minimize the error between estimated and real prices (least squares method). Prices are on a logarithmic scale, so each weight represents a percentage effect: for example, each more recent release year increases the value by about 2.7%.

### Feature selection
The features were chosen through a series of experiments, comparing the R² score of each combination:

| Experiment | R² |
|---|---|
| Base (new price, release year, days used, RAM) + battery + 5G | 0.786 |
| Internal memory instead of battery | 0.779 |
| + cameras | 0.804 |
| 4G instead of 5G | 0.803 |
| No 4G/5G (simpler, same score) | 0.803 |
| **+ screen size (final model)** | **0.820** |

Some of my initial assumptions were disproved by the data: camera megapixels, for example, improve the estimate, because they indicate the price range of the phone.

### Validation
* **Test on data:** the model explains 82% of the price differences (R² = 0.82) on 599 phones never seen during training.
* **Test against experience:** the estimates are consistent with the depreciation I see every day in the store. For example, a 300 € phone after about a year of use is estimated at 124 €, a little less than half of its new price.


## Challenges

What the project does **not** solve:
* **Physical condition:** the model knows nothing about scratches, damage or battery wear, which weigh heavily in real trade-ins.
* **2021 data:** prices do not reflect today's market, and the most recent phones are missing.
* **High-end phones:** the dataset contains very few phones above 500 €, so estimates in this range are unreliable (the model tends to underestimate them).
* **The model must be known:** to enter battery, cameras and screen size you need to know which phone is being valued. If the customer doesn't know, the tool cannot help.
* **Foldables excluded:** the 7" filter removes foldable phones together with tablets.

Things to keep in mind when reading the results:
* **Correlation, not causation:** the model finds relationships in the data, not the reasons behind them. Battery and cameras matter not because customers pay more for them, but because they indicate the phone's price range.
* **Weights must be read together:** some features overlap. For example, battery and screen size go hand in hand, because bigger phones have bigger batteries. When tablets were still in the dataset, adding screen size brought the battery weight down to almost zero: the two features carried the same information. On phones only, they overlap just partially, and together they improve the estimate (R² from 0.803 to 0.820).
* **Small R² differences** (below 0.01) depend on the random split between training and test data, and should not be read as real improvements.

Ethical aspect: an estimate communicated with too much confidence could create wrong expectations in the customer. For this reason it should always be presented as indicative, never as an offer.


## What next?

* **Compare with other methods:** try nearest neighbor and check whether it estimates better than linear regression.
* **Simple interface:** a web page where the salesperson picks the model from a menu and the technical features are filled in automatically.
* **Physical condition:** add a wear rating (excellent / good / damaged) and, later, estimate it from a photo with computer vision.
* **Updated Italian data:** collect recent used smartphone prices from public sources, to get estimates that are valid today.
* **Compare with an LLM:** compare the model's estimates with those of a language model with access to up-to-date prices, evaluating both accuracy and transparency.

To move forward, the project would need web development skills for the interface, computer vision skills for photo analysis, and above all access to real and up-to-date trade-in data.


## Acknowledgments

* Dataset: [Used Phones & Tablets Pricing Dataset](https://www.kaggle.com/datasets/ahsan81/used-handheld-device-data) by Ahsan Raza on Kaggle, license [CC0: Public Domain](https://creativecommons.org/publicdomain/zero/1.0/)
* [Building AI](https://buildingai.elementsofai.com/) course by Reaktor and the University of Helsinki
* Python libraries: pandas, NumPy, scikit-learn
* Developed with the support of Claude (Anthropic) as an assistant for the code and for explaining the concepts

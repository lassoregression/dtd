# dtd

<h1>Measuring Distance to Default (DTD) for Risk Management (Py)</h1>


<h2>Objective</h2>

To compute DTD using financial data <em>(from 2020-2022)</em>, applying theory from the paper;

[Measuring Distance-to-Default for Financial and Non-Financial Firms](http://d.nuscri.org/static/pdf/Measuring%20DTD_GCR_2012.pdf)

Distance-to-Default (DTD), a popular measure for gauging how far a limited-liability firm is away from default.

The analysis focuses on <b>IBM</b>, <b>General Motors (GM)</b>, and <b>Silicon Valley Bank (SVB)</b>, with two models implemented:
- Market Value Proxy Method
- Volatility Restriction Method

<h2>Assumptions</h2>

- The information was reported based on the date it was disclosed rather than the quarter-end date for updating book values, as the market was unaware of the information until the reporting date, and any subsequent changes in equity would only occur after it was reported.
- All the debt was considered current, except for the long-term debt associated with SVB.

<h2>Valuation Challenge</h2>

- Financial firms often exhibit inflated asset volatility due to their substantial liabilities, causing distortions in standard KMV DTD modeling.

<b>Stabilizing DTD Estimates:</b> To reduce sampling errors in DTD calculations, <b>Equation (5)</b> <em>(from the paper)</em> is applied, which sets the mean equal to sigma squared divided by two, providing a more stable estimate of DTD.

<h1>Market Value Proxy Method</h1>

Asset value = Market Cap of Equity (Daily) + Book Value of Liabilities (Quarterly)

<em>Estimated the Mean and Volatility of the Asset from the above equation. </em>

<sub><em>P.S The book value of liabilities was sourced from the consolidated balance sheet on a quarterly basis.  
For the purpose of this analysis, total debt is calculated as the sum of current debt + 50% of long-term debt. Missing data points were handled with forward fill.</em></sub>


The DTD was computed from <b>Equation (4)</b> and the DTD* from <b>Equation (5)</b>.

Specifically, utilizing the Market Value Proxy Method, the estimates closely aligned with expectations:  
SVB had the lowest DTD*, followed by IBM, and then GM.

An issue was noted with the initial DTD for IBM being negative, significantly influenced by the (mu) parameter.

<b>Equation 4</b> (below) illustrates the influence of the growth factor. By eliminating the drift component, DTD becomes less sensitive to company performance and economic changes.

$$
\text{DTD}_t = \frac{\ln \left( \frac{V_t}{F} \right) + \left( \mu - \frac{\sigma^2}{2} \right)(T - t)}{\sigma \sqrt{T - t}}
$$  


<b>Equation 5</b> (below) refines the estimation of Distance to Default (DTD) by adjusting for asset volatility and the time remaining until debt maturity, which enhances the model’s precision and stability.

$$
\text{DTD}^*_t = \frac{\ln \left( \frac{V_t}{F} \right)}{\sigma \sqrt{T - t}}
$$

This formula calculates the normalized log of the ratio of the firm’s asset value (Vt) to its debt (F) at time (t), divided by the product of asset volatility (σ) and the square root of the time to maturity (T-t)  

It reduces the influence of short-term fluctuations in asset values on the DTD calculation, focusing on long-term financial stability and providing a more stable estimate of default risk. Removes the drift element in the calculation and focuses solely on the asset volatility and the
leverage ratio.

<em>Due to the 5-quarter sample period and short horizon, a simpler model for the DTD is preferred.</em>

## Results (Market Value Proxy Method)

| Company | Mean (mu) | Volatility (sigma) | DTD   | DTD*   |
|---------|------------|--------------------|-------|--------|
| IBM     | -0.0920    | 0.1228             | -0.7582 | 0.0528 |
| GM      | 0.1013     | 0.1028             | 1.0210  | 0.0864 |
| SVB     | 0.0841     | 0.1017             | 0.7845  | 0.0090 |

<h2>Potential Limitations (Market Value Proxy Method)</h2>

- Produces an upward biased estimate of the asset value related to how volatile the asset is. This is due to the value of the discounted debt being set to par, which increases the market value and the firm’s volatility is increased.
- Firm’s equity is always in the money at the time of assessing the credit risk, this would produce an upward biased estimate of the DTD making default probability smaller.
- Potential issues with the standard deviation of positive asset returns affecting the asset volatility. A different treatment of the positive and negative returns may be necessary.

<h1>Volatility Restriction Method</h1>

A popular way of implementing the Merton (1974) model for pricing corporate bonds and other credit sensitive instruments is the volatility restriction method of Jones et al. (1984) and Ronn and Verma (1986).  

The volatility restriction method uses the following two-equation system:

$$
S_t = S(V_t; \sigma)
$$

$$
\sigma_{s_t} = \sigma \frac{V_t}{S(V_t; \sigma)} N(d_t)
$$

<em>Note: Premise: Equity as a Call Option</em>

The two-equation system is to link observed market capitalization to its theoretical counterpart and equate equity volatility with asset volatility, using Ito's lemma from Black-Scholes pricing.

Equation (7) forms a volatility restriction linking the equity volatility to the asset volatility where its right hand side can be derived by applying Ito’s lemma to the pricing formula in Equation (2). <em>(Refer to the paper for more details)</em>

**Market Cap and Theoretical Asset Value:**

The following equation links market cap with theoretical asset value:

$$
S_t = S(V_t; \sigma)
$$

**Volatility Restriction:**

The following equation establishes a volatility constraint by relating equity volatility to asset volatility:

$$
\sigma_{s_t} = \sigma \frac{V_t}{S(V_t; \sigma)} N(d_t)
$$


**Method 1:** Repeatedly solve the system to derive asset values over time, calculating the mean of the derived returns.  
**Method 2 (_Preferred_):** Solve the system once at a specific point, then apply the derived asset volatility historically to estimate asset values. The growth rate (mu) is obtained from the asset value time series. Finally, calculate DTD/DTD*.

**Given:**
- Equity Volatility, Market Cap, and Value of the Debt.
- Utilized the `brentq` root-finding algorithm from the `scipy` Python package to iteratively solve for the Asset Value time series by zeroing the difference between the theoretical and market value of the equity.
- Used the relevant 1-year T-bill rate from FRED as the risk-free rate.

**Procedure:**

- **Derive Asset Volatility:** Asset volatility is derived from equity volatility at \( T = 1 \) year.

$$
\sigma_S = \sigma_V \cdot \left( \frac{V}{S} \right) \cdot \Phi(d_1)
$$

- **Implied Asset Value:** Apply the derived asset volatility to obtain the implied asset value for \( t < T \).
- **Growth Rate Calculation:** Compute the growth rate (μ) from the implied asset value time series.


## Results (Volatility Restriction Method)

| Company | Asset Value      | Mean (mu)  | Volatility (sigma) | DTD     | DTD*    |
|---------|------------------|------------|---------------------|---------|---------|
| IBM     | 103711451976.6060 | -0.1683    | 5.1054              | -2.4957 | 0.0900  |
| GM      | 846365903.8802    | 0.0656     | 1.1729              | 0.0300  | 0.5606  |
| SVB     | 179414246.3611    | 0.0435     | 1.2236              | -0.0624 | 0.5139  |

<h2>Potential Limitations (Volatility Restriction Method)</h2>

- According to the Merton (1974) model, the derived equity volatility must be a stochastic variable.
- Since it is not a parameter, the sample standard deviation of equity returns should not be the quantity being plugged into the left-hand side of Equation (7).
- In practice, one can still obtain estimates but abusing the system could produce seriously biased estimates.

<h2>Comparison of DTD & DTD* Values Across Methods</h2>

<p align="center">
The following graphs present a comparison of Distance to Default (DTD) and Adjusted Distance to Default (DTD*) for <em>IBM, General Motors (GM), and Silicon Valley Bank (SVB)</em> using two methodologies: <b>Market Value Proxy Method and Volatility Restricted.</b>
<br>
  <br/>
<img src="https://i.imgur.com/3VgtQsU.png" height="80%" width="80%" alt="dtdcomp"/>
<br />
<br />

<h2>Data Analysis</h2>

- <h3>Market Value Proxy Method</h3>

- IBM shows a negative DTD of -0.7582, suggesting a very high risk of default within the next
year under this model. The DTD∗ value is low at 0.0528, indicating high volatility and potential
financial instability.
- GM has a positive DTD of 1.0210, placing it further from default risk compared to IBM. Its DTD∗
is higher at 0.0864, showing less volatility and a relatively healthier financial state.
- SVB also displays a positive DTD of 0.7845 with a very low DTD∗ of 0.0090, suggesting that
while the risk of default is less imminent than IBM, the asset value’s volatility is minimal,
indicating stability.

- <h3>Volatility Restricted</h3>

- IBM exhibits a significantly negative DTD of -2.4957, indicating an extremely high default risk.
The DTD∗ at 0.0900 reflects substantial volatility.
- GM shows a very marginal DTD of 0.0300, almost neutral, suggesting negligible default risk
under this model but still on the brink. Its DTD∗ of 0.5606 suggests higher volatility.
- SVB presents a negative DTD of -0.0624, indicating a slight risk of default, with a relatively high
DTD* of 0.5139, signifying notable volatility.

<h2>Model Comparisons</h2>

<h3>Conservatism</h3>
The Volatility Restricted Method appears more conservative, especially with IBM and SVB, where it projects a significantly higher risk of default (more negative DTDs). This method seems to take a more cautious approach by potentially considering more risk factors or using a different risk adjustment mechanism.

<h3>Simplicity</h3>
The Market Value Proxy Method is simpler as it uses direct market values and straightforward calculations of liabilities to assess the default risk. This method tends to provide more moderate assessments of default risk, as seen with less negative or more positive DTD values.

<h1>Conclusion</h1>

Based on the calculated DTD and DTD∗ values, it is clear that the choice of model can significantly influence the perceived financial stability of a company.  


The Volatility Restricted Method, by showing more negative DTD values, emphasizes a more cautious evaluation, potentially making it suitable for scenarios where a conservative approach is necessary.  
In contrast, the Market Value Proxy Method provides a clearer picture of immediate financial health without the extensive considerations of the underlying asset volatility, making it preferable for straightforward financial analysis.


In practice, the choice between these models would depend on the risk tolerance of the stakeholders and the specific financial analysis needs of the situation. For firms like IBM, with higher volatility and risk as shown in the data, close monitoring and perhaps more conservative financial planning might be advisable. For GM and SVB, while they show better stability, the choice of model might affect strategic decisions differently, especially in terms of capital structure and risk management strategies.

# Academic References

Research foundations underlying DROS design decisions. Each reference is linked to a specific system component.

---

## Machine Learning and Statistics

### Backtesting and Validation

**Lopez de Prado, M. (2018)**. *Advances in Financial Machine Learning*. Wiley.
- CPCV (Combinatorial Purge Cross-Validation): purge gap >= 24h requirement (INVARIANT-LEARNING-19)
- PBO (Probability of Backtest Overfitting): threshold < 0.3 gate (FAIL_PBO_OVERFIT)
- Embargo periods for time-series cross-validation

**Bailey, D.H., et al. (2014)**. *The Probability of Backtest Overfitting*. Journal of Computational Finance.
- PBO statistical foundation for A8 ROIDebias validation gate

### Calibration

**Niculescu-Mizil, A. & Caruana, R. (2005)**. *Predicting Good Probabilities with Supervised Learning*. ICML.
- Isotonic Regression for probability calibration (SparseSafeCalibrator > 500 samples)

**Kull, M., et al. (2017)**. *Beta calibration: a well-founded and easily implemented improvement on logistic calibration for binary classifiers*. AISTATS.
- Beta Calibration for 50-500 samples (SparseSafeCalibrator)

**Guo, C., et al. (2017)**. *On Calibration of Modern Neural Networks*. ICML.
- Temperature Scaling for < 50 samples (SparseSafeCalibrator)

### Reinforcement Learning

**Peng, X.B., et al. (2019)**. *Advantage-Weighted Regression: Simple and Scalable Off-Policy Reinforcement Learning*. arXiv:1910.00177.
- AWR Grid Agent: off-policy RL replacing PPO

**Thompson, W.R. (1933)**. *On the likelihood that one unknown probability exceeds another*. Biometrika.
- Thompson Sampling for symbol+regime exploration-exploitation

### Sequential Testing

**Johari, R., et al. (2022)**. *Always Valid Inference: Continuous Monitoring of A/B Tests*. Operations Research.
- Truncated mSPRT for evidence-based graduation gate (SPA p < 0.01)
- Maintains type-I error under optional stopping

### Bayesian Methods

**Gelman, A., et al. (2013)**. *Bayesian Data Analysis*, 3rd ed. CRC Press.
- Hierarchical Bayesian debias: symbol -> cluster -> global (INVARIANT-LEARNING-10)
- Empirical Bayes kappa estimation from marginal likelihood (INVARIANT-LEARNING-18)

---

## Behavioral Finance and Psychology

**Kahneman, D. & Tversky, A. (1979)**. *Prospect Theory: An Analysis of Decision under Risk*. Econometrica.
- Loss aversion framing in marketing copy: "Expansion paused" over "missed gain"
- Dead Zone design: abstain rather than force marginal decisions

**Cialdini, R. (1984)**. *Influence: The Psychology of Persuasion*.
- Social proof in transparent observation publishing
- Lead time as verifiable credibility signal

**Zeigarnik, B. (1927)**. *On Finished and Unfinished Tasks*. Psychologische Forschung.
- SYMBOL_WATCH open loop -> SYMBOL_CONFIRMED closure
- Observation window creates attention persistence

**Nosek, B.A., et al. (2018)**. *The preregistration revolution*. PNAS.
- Registered Reports model: method fixed before results
- WATCH filed before outcome -> CONFIRMED/EXPIRED on same ledger

---

## Market Microstructure

**Easley, D., et al. (2012)**. *Flow Toxicity and Liquidity in a High-Frequency World*. Review of Financial Studies.
- VPIN (Volume-synchronized Probability of Informed trading) for toxicity detection
- Game Theory layer: order flow analysis (INVARIANT-GT-01)

**Kyle, A.S. (1985)**. *Continuous Auctions and Insider Trading*. Econometrica.
- Informed trader detection framework underlying SmartMoney axis

**Amihud, Y. (2002)**. *Illiquidity and stock returns*. Journal of Financial Markets.
- Liquidity adjustment in position sizing

---

## Volatility Estimation

**Yang, D. & Zhang, Q. (2000)**. *Drift-Independent Volatility Estimation Based on High, Low, Open, and Close Prices*. Journal of Business.
- Yang-Zhang volatility estimator: primary sigma_d for spacing computation
- ATR-only sigma_d estimation banned (INVARIANT-SPACING-04)

**Rogers, L.C.G. & Satchell, S.E. (1991)**. *Estimating Variance from High, Low and Closing Prices*. Annals of Applied Probability.
- Range-based volatility estimation foundation

---

## Change Detection

**Adams, R.P. & MacKay, D.J.C. (2007)**. *Bayesian Online Changepoint Detection*. arXiv:0710.3742.
- BOCPD: one of four detectors in Black Swan ensemble (USE_EVOL_BLACK_SWAN)

**Hawkes, A.G. (1971)**. *Spectra of some self-exciting and mutually exciting point processes*. Biometrika.
- Hawkes process for event clustering in Black Swan ensemble

**Bifet, A. & Gavalda, R. (2007)**. *Learning from Time-Changing Data with Adaptive Windowing*. SIAM International Conference on Data Mining.
- ADWIN for adaptive change detection

---

## Transparency and Trust

**Numerai (ongoing)**. Transparency model: publishing all predictions including failures.
- EXPIRED watch signals never suppressed (INVARIANT-MKT-29)
- Confirmation rate meaningful only when all outcomes reported

**U.S. Securities and Exchange Commission (2021)**. *Marketing Rule for Investment Advisers* (17 CFR 275.206(4)-1).
- No performance guarantees in public communications
- Fair, clear, not misleading standard for all channel content

**European Union (2023)**. *Markets in Crypto-Assets Regulation (MiCA)*.
- Clear disclosure requirements for automated trading communications
- Methodology documentation requirements (docs/open-architecture.md)

---

## Algorithm Aversion and Trust

**Dietvorst, B.J., et al. (2015)**. *Algorithm aversion: People erroneously avoid algorithms after seeing them err*. Journal of Experimental Psychology: General.
- Referenced in marketing psych_policy.py: weak control (Bot query commands) reduces aversion
- INFORMS mnsc.2016.2643: partial control improves algorithmic acceptance

**Plain Language Guidelines (Digital.gov)**. *Federal Plain Language Guidelines*.
- Public channel copy: first sentence comprehensible without domain knowledge
- Technical terms maximum 1 per first line (telegram_public COMPREHENSION_LEVEL)

---

## Quality-Diversity Optimization

**Mouret, J.B. & Clune, J. (2015)**. *Illuminating search spaces by mapping elites*. arXiv:1504.04909.
- MAP-Elites foundation for AlphaFoundry QD genome evolution
- Quality (net ROI) x Diversity (regime x symbol_tier coverage) optimization

---

*References for DROS v12.4b | 2026-03-14*

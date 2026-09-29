# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
### Chain Design Visualization
```mermaid
flowchart LR
    A[List of receipt images from public_test folder] --> B[Read each image and encode to base64]
    B --> C[Vision prompt + deepseek-v4-flash-vision-exp]
    C --> D[Vision model reads single receipt]
    D --> C1[Extract: actual paid amount after rounding]
    D --> C2[Extract discounted subtotal]
    D --> C3[Extract all negative discount/promotion/coupon entries above subtotal]
    C3 --> E[Calculate sum of absolute values of deduction]
    C2 & E --> F[pre_deduction_total = subtotal + sum of absolute values of deductions]
    C1 & F --> G[Output: paid amount, pre_deduction_total for one receipt]
    G --> H[Sum values across all receipts]
    H --> I[Return two final total HKD results]
```

### **SOLUTION DESCRIPTION**
This solution uses deepseek-v4-flash-vision-exp as the vision model. I take the list of receipt images from the public_test folder, read each image and convert it into base64 format as the model input. For every receipt, the model extracts key pieces of information: the final actual paid amount, the discounted subtotal printed on the receipt, and all negative deduction lines above the subtotal. Instead of summing every individual product's original price, the pre-deduction total is calculated by adding the discounted subtotal to the sum of absolute values of those negative deduction entries. The model outputs two decimal numbers for each receipt. These numbers are converted to Decimal type and sum over all receipts to produce the two final predicted HKD totals for the questions.

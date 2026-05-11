### in the name of Allah
# RecommenderSystemUsingARM - ECWH3
>>> Electronic Commerce - Winter 2025

--------------------------------------------------------------------------------------------------------------
## TASK I: Data Preprocessing

This task focuses on preparing the Instacart e-commerce dataset for Association Rule Mining analysis. The preprocessing pipeline ensures data quality and creates a manageable subset for efficient pattern mining. We address common data issues including missing values and irrelevant transactions, while focusing on multi-item baskets essential for meaningful association rule discovery. By sampling 20,000 users, we balance computational feasibility with sufficient data coverage for robust analysis.

### **Step 1: Loading the Data**
Load CSV files:
- **pandas** for small files: `aisles.csv`, `departments.csv`, `products.csv`
- **dask** for large files: `order_products__train.csv`, `orders.csv`, `order_products__prior.csv`

### **Step 2: Data Cleaning Functions**
Define functions:
- **`remove_nulls(df)`**: Remove rows with missing values.
- **`filter_single_item_orders(df)`**: Remove orders with only one product.
- **`filter_by_order_ids(df, order_ids_set)`**: Filter by order IDs.

### **Step 3: Preprocessing Execution**
1. **Remove nulls** from all tables.
2. **Remove single-item orders** from order products data.
3. **Sample 20,000 users** randomly from orders.
4. **Extract orders** for sampled users.
5. **Filter order products** to include only sampled orders.

### **Step 4: Saving Cleaned Data**
Save cleaned data to `./processed_data/`:
- **`aisles_cleaned.csv`**, **`products_cleaned.csv`**, **`departments_cleaned.csv`**
- **`orders_sampled.csv`**: Orders of 20,000 sampled users.
- **`order_products_train_sampled.csv`**, **`order_products_prior_sampled.csv`**
- **`order_products_combined.csv`**: Combined data for basket analysis.

This preprocessing ensures clean, multi-item basket data ready for Association Rule Mining in subsequent tasks.

--------------------------------------------------------------------------------------------------------------

## TASK II: Basket Encoding for Association Rule Mining

This task transforms the cleaned transactional data into a binary matrix format suitable for Association Rule Mining algorithms like Apriori. We convert order-level product lists into a one-hot encoded representation, focusing on frequently purchased products to reduce dimensionality and computational complexity. The process ensures that only meaningful multi-item baskets are retained for pattern discovery.

### **Step 1: Loading Processed Data**
Load the combined order products data and product mappings from Task 1:
- `order_products_combined.csv`: Contains order-product pairs.
- `products_cleaned.csv`: Provides product ID to name mappings.

### **Step 2: Creating Shopping Baskets**
Group products by order ID to form shopping baskets:
- Each basket represents a list of products purchased together in a single order.
- This step converts transactional data into a basket format essential for market basket analysis.

### **Step 3: One-Hot Encoding & Product Filtering**
1. **Identify Unique Products:** Extract all unique products across baskets.
2. **Filter Frequent Products:** Retain only products that appear in at least 0.5% of baskets to focus on significant items and reduce noise.
3. **Build Binary Matrix:** Create a boolean matrix where rows correspond to baskets and columns to filtered products. A value of `True` indicates the presence of a product in a basket.
4. **Remove Sparse Baskets:** Discard baskets with fewer than 2 products to ensure meaningful associations.

### **Step 4: Saving the Encoded Matrix**
- Save the final binary matrix as `basket_encoded.csv` for use in subsequent association rule mining tasks. This matrix optimizes memory usage and computational efficiency for the Apriori algorithm.
- This encoding step transforms raw transactional data into a structured format ready for frequent itemset mining and association rule generation.

--------------------------------------------------------------------------------------------------------------

## TASK III: Frequent Itemset Mining with Apriori Algorithm

This task implements the Apriori algorithm to discover frequent itemsets from the encoded basket data. We systematically explore different minimum support thresholds to understand their impact on the mining results, enabling data-driven parameter selection for optimal association rule discovery.

### **Step 1: Parameter Exploration**
Execute Apriori algorithm across multiple minimum support values (0.05 to 0.01) to analyze sensitivity:
- **Higher support (0.05-0.035)**: Produces only single-item sets (14-25 itemsets).
- **Critical threshold (0.03)**: First appearance of 2-item sets (39 itemsets).
- **Optimal range (0.025-0.01)**: Balanced quantity and quality (59-212 itemsets).

### **Step 2: Visual Analysis**
Generate comparative visualizations:
- **Total Itemsets vs Minimum Support**: Inverse relationship showing exponential growth in itemset count as support decreases.
- **Maximum Itemset Size vs Minimum Support**: Step function revealing the support threshold (0.03) where 2-item sets emerge.

![min support vs frequent itemset size plots](results/apriori_comparison.png)

### **Step 3: Optimal Parameter Selection**
Based on analysis, select min_support = 0.025 as optimal balance point:
1. **59 frequent itemsets** identified
2. **Size distribution**: 54 single-item sets (91.5%) + 5 two-item sets (8.5%)
3. **Top performers**: Product_24852 (20.94% support), Product_13176 (17.01%), Product_21137 (12.43%)

### **Step 4: Results Storage**
- Save complete frequent itemsets to `frequent_itemsets_0025.csv` for use in association rule generation. This dataset forms the foundation for deriving meaningful product association rules in Task 4.
- This systematic parameter exploration ensures robust selection of mining parameters, balancing computational efficiency with comprehensive pattern discovery for actionable business insights.
--------------------------------------------------------------------------------------------------------------

## TASK IV: Association Rule Mining and Analysis

This task generates and analyzes association rules from the frequent itemsets discovered in Task 3. Using the mlxtend library's `association_rules` function, we extract meaningful relationships between products and evaluate them using key metrics including support, confidence, and lift. The analysis identifies the strongest product associations for actionable business recommendations.

### **Step 1: Loading Frequent Itemsets**
Load the precomputed frequent itemsets from Task 3:
- `frequent_itemsets_0025.csv`: Contains 59 frequent itemsets with min_support=0.025.
- Convert string representations of itemsets back to Python sets for rule generation.
- Itemset size ranges from 1 to 2, enabling discovery of pairwise product relationships.

### **Step 2: Generating and Analyzing Association Rules**
Apply the association rules algorithm with minimum confidence threshold of 0.1:
- **10 association rules** generated from the frequent itemsets.
- **All rules have lift > 1**, indicating positive relationships between products.
- **Support range**: 0.0251 to 0.0302 (2.5% to 3% of transactions).
- **Confidence range**: 0.1197 to 0.3059 (12% to 31% prediction accuracy).
- **Average lift:**: 1.358, showing moderate to strong associations.

### **Step 3: Identifying Top Rules by Lift**
Extract the three strongest association rules based on lift value:
1. **Product_13176 → Product_47209** (Lift: 1.7398, Confidence: 17.13%, Support: 2.91%)
2. **Product_47209 → Product_13176** (Lift: 1.7398, Confidence: 29.59%, Support: 2.91%)
3. **Product_47766 → Product_24852** (Lift: 1.4604, Confidence: 30.59%, Support: 2.51%)

### **Step 4: Saving Results and Final Summary**
Save the complete analysis for business implementation:
- `association_rules.csv`: All 10 generated rules with full metrics.
- `top_3_rules_by_lift.csv`: Filtered set of strongest rules for immediate action.
- **Key insight**: The bidirectional relationship between Product_13176 and Product_47209 represents the strongest association in the dataset.

This association rule mining provides actionable insights for product placement, promotional bundling, and cross-selling strategies, enabling data-driven decision making for retail optimization.

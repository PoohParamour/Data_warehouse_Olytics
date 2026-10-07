# Brazilian E-Commerce Public Dataset by Olist - Knowledge Base

## Overview
This is a Brazilian e-commerce public dataset of orders made at the Olist Store. The dataset contains information on 100k orders from 2016 to 2018 made at multiple marketplaces in Brazil. Its features allow viewing an order from multiple dimensions: from order status, price, payment, and freight performance to customer location, product attributes, and reviews written by customers. It also includes a geolocation dataset that relates Brazilian zip codes to lat/lng coordinates.

This is real commercial data that has been anonymized. References to companies and partners in the review text have been replaced with the names of Game of Thrones great houses.

**Marketing Funnel Integration:**
Olist has also released a [Marketing Funnel Dataset](https://www.kaggle.com/olistbr/marketing-funnel-olist/home). You may join both datasets to see an order from the marketing perspective.

## Context
Olist is the largest department store in Brazilian marketplaces. It connects small businesses from all over Brazil to channels seamlessly. Merchants sell their products through the Olist Store and ship them directly to customers using Olist logistics partners.

**Key characteristics:**
- An order might have multiple items.
- Each item might be fulfilled by a distinct seller.

## Data Schema & Relationships

The dataset is structured relationally, similar to a star/snowflake schema. Here is the detailed breakdown of tables and their connections based on the `data` directory files:

### 1. `olist_orders_dataset.csv` (Central Fact Table)
Contains the core information of each order.
- **Columns:**
  - `order_id`: (Primary Key) Unique identifier for the order.
  - `customer_id`: (Foreign Key) Links to `olist_customers_dataset.customer_id`. Note: this is unique per order, use `customer_unique_id` in the customers table to track identical customers across multiple orders.
  - `order_status`: Status of the order (e.g., delivered, shipped).
  - `order_purchase_timestamp`: Purchase date.
  - `order_approved_at`: Payment approval date.
  - `order_delivered_carrier_date`: Date handled to the logistics partner.
  - `order_delivered_customer_date`: Actual delivery date.
  - `order_estimated_delivery_date`: Estimated delivery date given to the customer.

### 2. `olist_customers_dataset.csv`
Information about the customer and their location.
- **Columns:**
  - `customer_id`: (Primary Key) Key to the orders dataset. Each order has a unique `customer_id`.
  - `customer_unique_id`: Unique identifier of a customer. Used to identify customers that made repurchases.
  - `customer_zip_code_prefix`: (Foreign Key) First five digits of customer zip code. Links to `olist_geolocation_dataset.geolocation_zip_code_prefix`.
  - `customer_city`: Customer city name.
  - `customer_state`: Customer state.

### 3. `olist_order_items_dataset.csv`
Contains details of the items purchased within each order.
- **Columns:**
  - `order_id`: (Foreign Key) Links to `olist_orders_dataset`.
  - `order_item_id`: Sequential number identifying number of items included in the same order.
  - `product_id`: (Foreign Key) Links to `olist_products_dataset`.
  - `seller_id`: (Foreign Key) Links to `olist_sellers_dataset`.
  - `shipping_limit_date`: Seller shipping limit date.
  - `price`: Item price.
  - `freight_value`: Item freight value item (if an order has more than one item the freight value is split between items).

### 4. `olist_products_dataset.csv`
Contains data about the products sold.
- **Columns:**
  - `product_id`: (Primary Key) Unique product identifier.
  - `product_category_name`: Root category of product (in Portuguese). Can be joined with `product_category_name_translation.csv` for English.
  - `product_name_lenght`: Number of characters extracted from the product name.
  - `product_description_lenght`: Number of characters extracted from the product description.
  - `product_photos_qty`: Number of product published photos.
  - `product_weight_g`: Product weight measured in grams.
  - `product_length_cm`: Product length measured in centimeters.
  - `product_height_cm`: Product height measured in centimeters.
  - `product_width_cm`: Product width measured in centimeters.

### 5. `olist_sellers_dataset.csv`
Contains data about the sellers that fulfilled orders.
- **Columns:**
  - `seller_id`: (Primary Key) Seller unique identifier.
  - `seller_zip_code_prefix`: (Foreign Key) Links to `olist_geolocation_dataset.geolocation_zip_code_prefix`.
  - `seller_city`: Seller city name.
  - `seller_state`: Seller state.

### 6. `olist_order_payments_dataset.csv`
Contains data about the payment methods used for the orders.
- **Columns:**
  - `order_id`: (Foreign Key) Links to `olist_orders_dataset`.
  - `payment_sequential`: A customer may pay an order with more than one payment method. If so, a sequence is created to accommodate all payments.
  - `payment_type`: Method of payment (e.g., credit_card, boleto).
  - `payment_installments`: Number of installments chosen by the customer.
  - `payment_value`: Transaction value.

### 7. `olist_order_reviews_dataset.csv`
Contains data about the reviews made by customers.
- **Columns:**
  - `review_id`: (Primary Key) Unique review identifier.
  - `order_id`: (Foreign Key) Links to `olist_orders_dataset`.
  - `review_score`: Note ranging from 1 to 5 given by the customer on a satisfaction survey.
  - `review_comment_title`: Comment title from the review left by the customer.
  - `review_comment_message`: Comment message from the review left by the customer.
  - `review_creation_date`: Date in which the satisfaction survey was sent to the customer.
  - `review_answer_timestamp`: Timestamp of the satisfaction survey answer.

### 8. `olist_geolocation_dataset.csv`
Information about Brazilian zip codes and their lat/lng coordinates.
- **Columns:**
  - `geolocation_zip_code_prefix`: First five digits of zip code.
  - `geolocation_lat`: Latitude.
  - `geolocation_lng`: Longitude.
  - `geolocation_city`: City name.
  - `geolocation_state`: State name.

### 9. `product_category_name_translation.csv`
Translates the product category name to english.
- **Columns:**
  - `product_category_name`: Category name in Portuguese.
  - `product_category_name_english`: Category name in English.

## Best Practices for Usage
- **Customer Repurchases:** Do not use `olist_orders_dataset.customer_id` to track a single customer's repurchases. Always join with `olist_customers_dataset` and group by `customer_unique_id`.
- **Item Level vs Order Level Analysis:** When analyzing revenue, be mindful of the granularity. `olist_order_items_dataset` is at the item level. Group by `order_id` to aggregate item values when joining back to `olist_orders_dataset`.
- **Location Mapping:** Both customers and sellers have a `zip_code_prefix` which can be joined with the `olist_geolocation_dataset` to map distances, delivery times vs geography, etc.
- **Category Names:** For non-Portuguese speakers, remember to join `olist_products_dataset` with `product_category_name_translation.csv` to map `product_category_name` into English.

# Amazon_Vine_Analysis

## Project Overview

Amazon Vine is a program that allows manufacturers and publishers to receive product reviews. In this project, 
we aim to determine if there is a bias towards positive Vine reviews and assess the benefits for a company enrolling in such a program.


## Ressouces 

We are given access to approximately 50 datasets. Each one contains reviews of a specific product, 
from clothing apparel to wireless products:

[Amazon Review datasets](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt)

To do our analysis, we chose the dataset that contains reviews about multiple software:

[Project Dataset](https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Software_v1_00.tsv.gz)

With the chosen dataset, we use *PySpark*, a unified analytics engine for large-scale data processing, to 
perform the ETL process to extract the dataset and transform the data.

Then, we loaded the transformed data into *pgAdmin* connected to an *AWS PostgreSQL* instance. Finally, we also use *Pyspark* to determine
if there is any bias toward favorable reviews from Vine members in our dataset.


## Analysis results

### Perform ETL on Amazon Product Reviews using PySpark and PostgreSQL

The first step of our analysis is extracting the data from our chosen dataset.

The dataset we are analyzing, as with all other Amazon review datasets, has the following schemata:

*	marketplace       - 2-letter country code of the marketplace where the review was written.
*	customer_id       - Random identifier that can be used to aggregate reviews written by a single author.
*	review_id         - The unique ID of the review.
*	product_id        - The unique Product ID the review pertains to. In the multilingual dataset, the reviews
                    	for the same product in different countries can be grouped by the same product_id.
*	product_parent    - Random identifier that can aggregate reviews for the same product.
*	product_title     - Title of the product.
*	product_category  - Broad product category that can be used to group reviews 
                    (also used to group the dataset into coherent parts).
*	star_rating       - The 1-5 star rating of the review.
*	helpful_votes     - Number of helpful votes.
*	total_votes       - Number of total votes the review received.
*	vine              - The review was written as part of the Vine program.
*	verified_purchase - The review is on a verified purchase.
*	review_headline   - The title of the review.
*	review_body       - The review text.
*	review_date       - The date the review was written.

We are interested in the data in some columns, such as customer_id, review_id, and star_rating, to perform our analysis.

Before transforming the data, we created a new database with Amazon RDS to store the transformed data through pgAdmin.

The database we created contained the following tables and attributes:

* review_id table with the following attributes: 
	* review_id
	* customer_id
	* product_id
	* product_parent
* products table with the following attributes: 
	* product_id
	* product_title
* customers table with the following attributes: 
	* customer_id
	* customer_count
* vine table with the following attributes: 
	* review_id
	* helpful_votes
	* star_rating
	* total_votes
	* vine
	* verified_purchase

After we created the tables, we used PySpark to download the dataset into a DataFrame:

![download_dataset](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/dowload_dataset.png)

The dataFrame we created has the following schema:

![dataset_schema](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/data_set_schema.png)

Once we had the dataframe, we transformed it into the following four separate DataFrames that match the table schema in pgAdmin:

1.	The customers_table DataFrame
	*	To create this dataFrame, we used the groupby() function on the customer_id column, and to count all the customers, we used the 
		the agg() and count functions:
		![customer_dataframe](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/customer_table.png)
		
2.	The products_table DataFrame
	*	To create this dataFrame, we used the select() function to select the product_id and product_title, then drop 
		duplicates with the drop_duplicates() function to retrieve only unique product_ids.
		![product_dataframe](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/products_table.png)
	
3.	The review_id_table DataFrame
	*	To create the review_id_table, we used the select() function to select the columns that are in the review_id_table in pgAdmin,
		and converted the review_date column to a date using the to_date() function:
		![review_dataframe](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/review_table.png)
		![schema_review](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/to_date_result.png)
	
4.	The vine_table DataFrame	
	*	To create the vine_table, use the select() function to select only the columns that are in the vine_table in pgAdmin
		![vine_dataframe](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_table_df.png)


After all the dataFrames were created, we loaded the DataFrames that correspond to the tables in pgAdmin:

![customer_table_query](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/customer_table_query.png)

![product_table_query](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/product_table_query.png)

![review_table_query](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/review_table_query.png)

![vine_table_query](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_table_query.png)


### Determine the Bias of Vine Reviews

As $ellBy needs to determine if enrolling in a program like Amazon Vine is worth it, we
used PySpark and the data in the dataFrame we created for the vine table, which contains mainly
information regarding the reviews about the software, to perform the next part of our analysis.

To proceed, we created a new DataFrame in which we retrieved all the rows where the total_votes count 
is equal to or greater than 20 to pick reviews that are more likely helpful.

![vine_helpful](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_helpful_votes.png)

Using the new DataFrame, we retrieved all the rows where the number of helpful_votes divided by total_votes is equal 
to or greater than 50%.

![vine_helpful_total](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_helpful_total.png)

Then, we filtered the new dataFrame to retrieve all the rows where a review was written as part of the Vine program (paid):

![vine_paid](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_paid.png)

We repeated the last step to retrieve all the rows where a review was not part of the Vine program (unpaid):

![vine_unpaid](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/vine_unpaid_rev.png)

We used those four dataFrames to retrieve the following information:

* 	The total number number of paid reviews		
	- ![paid_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/paid_reviews.png)
	
*	The total number number of unpaid reviews
	- ![upaid_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/unpaid_reviews.png)
	
*	The total number of 5-star paid reviews
	- ![paid_5star_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/5star_paid.png)
	
*	The total number of 5-star unpaid reviews
	- ![paid_5star_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/5star_unpaid.png)
	
*	The percentage of 5-star reviews for paid reviews
	- ![percent_paid_5star_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/perc_5star_paid.png)
	
*	The percentage of 5-star reviews for unpaid reviews
	- ![percent_paid_5star_reviews](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/perc_5star_unpaid.png)
	
We created the following Pandas dataFrame to summarize the results and also plot them:

![PD_results](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/PD_results.png)

We plotted the number of paid and unpaid reviews against the number of paid and unpaid 5-star reviews:

![plot_paid_unpaid](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/plot_paid_unpaid.png)

And finally, we plotted the percentage of 5-star paid reviews against the percentage of unpaid 5-star reviews:

![polt_percentage](https://github.com/valerielnd/Amazon_Vine_Analysis/blob/main/plot_percentage.png)

Since the percentage of 5-star paid reviews is greater than the percent of 5-star unpaid reviews, while the number
of unpaid reviews is far greater than the number of paid reviews; there could be a positive bias for reviews in the Vine program.

One additional analysis we could do to dig further is to compare which software received the most 5-star
paid and unpaid reviews to see if they are mostly the same.





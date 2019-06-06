# /usr/bin/python

"""
Pet project for Home Depot product search from kaggle
Ref:  https://www.kaggle.com/c/home-depot-product-search-relevance
"""
from __future__ import print_function

if __name__ == '__main__':
    import sys
    import os
    import findspark
    findspark.init()
    from pyspark.sql import SparkSession
    from pyspark.sql import types
    from pyspark.sql import functions
    import pandas as pd

    # create new spark session
    spark = SparkSession.builder \
        .master("local") \
        .appName("spkJob") \
        .config("spark.sql.shuffle.partitions", "5")\
        .getOrCreate()

    # Default Directories and files:
    defaultDir = r'E:\rcnt\datadir'

print "********* App starts here ******************"
spark.sparkContext.setLogLevel("INFO")

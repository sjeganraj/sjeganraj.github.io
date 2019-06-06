# Apache Spark RDD notes:
                        
## Spark with python -- Transform,stage and store.

### Spark:
    -> Spark is in-memory distributed processing framework.
    It has multiple API's Spark-core,dataframe/SQL and Mlib api.

### Core-Spark API:

   RDD -- resilient distributed dataset :
        Which is a fault-tolerant collection of elements that can be operated on in parallel. There are two ways to create RDDs: parallelizing an existing collection in your driver program, or referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase, or any data source offering a Hadoop InputFormat.
    sc.parallelize(pythonListObj) -- Convert python list into spark RDD collection.


### RDD Actions:

    reduce(func)   -->	Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel.
    collect()  --> Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data.
    count()	   --> Return the number of elements in the dataset.
    first()	   --> Return the first element of the dataset (similar to take(1)).
    take(n)	   --> Return an array with the first n elements of the dataset.
    takeSample(withReplacement, num, [seed])   -->Return an array with a random sample of num elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed.
    takeOrdered(n, [ordering])  --> Return the first n elements of the RDD using either their natural order or a custom comparator.
    saveAsTextFile(path)    --> Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark                            will call toString on each element to convert it to a line of text in the file.
    saveAsSequenceFile(path) --> Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on                             RDDs of key-value pairs that implement Hadoop's Writable interface.
    saveAsObjectFile(path)  --> Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using SparkContext.objectFile().
    countByKey()    --> Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key.
    foreach(func)   --> Run a function func on each element of the dataset. This is usually done for side effects such as updating an Accumulator or interacting with external storage systems.

### Standard transformation:
    -> String Manipulation(Python):
        -- Split and get required fields.
        -- Data type conversion
        -- Discard unnecessary columns
        --

### Transformation:
    -- RDD Map:
    Ex: emp_dat = empRDD.map(lambda x: x.split(",")[5] )  -- Spilit date fileds from 5th position.
    emp_tup = empRDD.map(lambda x: (int(x.split("," )[0]),x.split("," )[1],x.split("," )[5],x.split("," )[6],int(x.split("," )[7]),int(x.split("," )[10]))) -- Convert RDD into Tuples (k,v).

    -- RDD flatMap:
    ex: flt = empRDD.flatMap(lambda x: x.splitlines()) # Flattens the RDD splitted lines.
    -- RDD filter:
    Ex:  fil = empRDD.filter(lambda x: x.split(",")[10]=='80')  # It will filter records and give dept_Id=80.

### Joins:
    -- Read text file using sc.textFile("file.csv") and convert as tuple using below stmts:
    -- empTab = empRDD.map(lambda x: (x.split("," )[10].strip(),int(x.split("," )[0]),x.split("," )[1],x.split("," )[5],x.split("," )[6],int(x.split("," )[7])))

    Ex:
    empTab = empRDD.map(lambda x: (x.split("," )[10].strip(),int(x.split("," )[0]),x.split("," )[1],x.split("," )[5],x.split("," )[6],int(x.split("," )[7])))

    deptTab = deptRDD.map(lambda x: (x.split("," )[0],x.split("," )[1]))

    empJoin = empTab.join(deptTab) # Join two RDD sets

    empLeftJoin = empTab.leftOuterJoin(deptTab) # Join two RDD sets

### Total Aggregation:
    count – give number of records in RDD
    reduce – used to perform aggregations such as sum, min, max etc on RDDs which contain numeric elements

    Ex: from operator impor add # Imort add module from operator
        empSal = empFile.map(lambda x: int(x.split(",")[7]))  # get salary field alone fro  emp csv file data.
        Tot = empSal.reduce(add) # will give total of all emp's salary.

    Get Min and Max using reduce function:
    Ex:
    max = empFile.reduce(lambda x,y: x if (x.split(",")[7])> y.split(",")[7] else y) # return X if X is greater than Y.

    min =  empFile.reduce(lambda x,y: x if (x.split(",")[7]) < y.split(",")[7] else y) # return X if X is less than Y.

### Aggregation by Key:
    countByKey:
        It is Aggregation and inout must be (k,v). and the result will return as python Dict type. Not spark RDD.

    Ex: cnt = empTab.countByKey()

### groupByKey:
    By using groupByKey can do aggregate functions. RDD must be (K,V) pair.

    Ex:
    empDetail = empFile.map(lambda x: (x.split("," )[10].strip(),(int(x.split("," )[7])))) # Get deptid,salary as K,V RDD
    salGroup = empDetail.groupByKey() # Use groupByKey fun
    perSal = salGroup.map(lambda  x: (x[0], sum (x[1]))) # Do aggregate fun over groupByKey.

## Transformations on Pair RDDs:
    1. reduceByKey(func)
    2. groupByKey()
    3. mapValues(func)
    4. flatMapVaues(func)
    5. combineByKey(createCombiner,mergeValue, mergeCombiners,partitioner)
    6. keys()
    7. values()
    8. sortByKey()

## Transformations on two pair RDDs:
    1. subtractByKey
    2. join
    3. rightOuterJoin
    4. leftOuterJoin
    5.cogroup

### Sample Code:
line_count = fileData.map(lambda line: line.splitlines())   #Count the no of lines
words = fileData.flatMap(lambda word: word.split(","))

count_desc = words.map(lambda word: (word, 1)) \
                  .reduceByKey(lambda a, b:  a + b) \
                  .sortBy(lambda x: x[1], False)    #Count the in descedning order

businessCount = fileData.map(lambda row: row.split(",")) \
                        .map(lambda line: (line[0], 1)) \
                        .reduceByKey(lambda a, b: a + b) \
                        .sortBy(lambda cnt: cnt[1], False)  # Business wise count in Match report data.

businessCount.saveAsTextFile(os.path.join(
    defaultDir, 'businessCount'))   # Write the coiunt data into os file.

### Sample streaming:

strm = spark.readStream.schema(strmSchema).format("csv").option(
        "header", "true").load(os.path.join(defaultDir, '*.csv'))

strm.writeStream.format("memory").queryName("vw_strm").start()

spark.sql("select * from vw_strm limit 10").show(5)

spark.sql("""
            SELECT StockCode, COUNT (*) AS cnt, ROUND (SUM (unitprice), 2) AS price
            FROM vw_strm
        GROUP BY StockCode
        ORDER BY cnt DESC """).show(15)

retailStream.isStreaming    # True will return if job is running.

This is my blogpost on assignment 3.

Assignment 3A focusses on the SPARK framework for Big Data. The assignment is divided into two, where part one main focus is RDD's and part two focusses on Dataframes and SPARK SQL. There is also assignment 3B, where the main goal is to analyze and work with real data of art and addresses in the city of Nijmegen and to integrate one into the other making use of SPARK SQL. 

In this blog, I will focus on part 3A, where I got an intro into RDD's and got to work with Dataframes in SPARK.

What is interesting in SPARK is that it is divided into 'Transformations' and 'Actions'. Spark makes use of lazy evaluation to save time and resources. Simply put, when a transformation is called on an RDD, the SPARK client will not execute it immediately. Instead, it will wait until an action is called on the RDD to perform the said transformations. This way, when multiple transformations are made before an action, the framework will optimize the chain of transformations. For example, in the assignment notebook, a chain of actions is called on the RDD 'words'.

	val words = lines.flatMap(line => line.split(" "))
	              .filter(_ != "")
	              .map(word => (word,1))
First, flatMap is called. Then, filter and map. The three transformations are chained together and the SPARK framework optimizes them. However, nothing happened yet because none of the commands are actions.

Further in the assignment, actions are used:

	wc.filter(_._1 == "Romeo").collect
	wc.filter(_._1 == "Julia").collect
The filter transformation transforms an RDD into one where the first word is "Romeo" or "Juliet". The action .collect then returns the word count for the respective words.

Another query in the SPARK notebook is:

	wc.filter(_._1 == "Macbeth").collect
and 

	val words = lines.flatMap(line => line.split(" "))
              .map(w => w.toLowerCase().replaceAll("(^[^a-z]+|[^a-z]+$)", ""))
              .filter(_ != "")
              .map(w => (w,1))
              .reduceByKey( _ + _ )
	words.filter(_._1 == "macbeth").collect
  		.map({case (w,c) => "%s occurs %d times".format(w,c)}).map(println)

Because the first query is not preprocessed for noise, it gives a different word count than the second query, where the words are preprocessed to be all lower case and where special characters are removed using regular expressions. This demonstrates the importance of understanding the data.

In the example, flatMap is used. The difference between map and flatmap seems to be that map maps to an array with the same dimensions, whereas in flatmap you can increase the dimensions of the resulting array. In the example, 'words' will be mapped onto a bigger array because the sentences are split into words using .split(" ").

One aspect of SPARK is that the data is held in memory. Using the command .cache, the data can be held in memory. This speeds up processing of future queries. There is also .persist. However, reading through the documentation of .cache and .persist, it seems like they do the same, just different syntax. In the notebook, the wordcount RDD wc is cached before filtering and collecting: 

	wc.cache()
	wc.filter(_._1 == "Macbeth").collect
	wc.filter(_._1 == "Capulet").collect
This speeds up processing of the transformations and the actions. It was explained in the lectures that caching occurs automatically in SPARK but that explicit use of the command can help the framework make the right decisions as to where to cache data. This makes sense, since in Big Data, the data is usually too big to fit in memory. Moreover, defining multiple RDD's means that it is potentially not possible to hold every RDD in memory. Reducing the RDD by filtering data helps the framework do its job properly. 

using 

	words.saveAsTextFile("wc")
the results are saved in multiple files. One of the questions is: Explain why there are multiple result files. This should be because the data is partitioned into multiple files and given to different workers. Each worker writes to its respective output file, creating multiple result files.

In the following notebook, it is explained that the standard number of partitions depends on the number of cores in the machine that is running docker. However, the user can specify the number of partitions:

	val rddRange = sc.parallelize(0 to 999,8)
Here, the user specified 8 partitioners.

The notebook makes use of a hashpartitioner. The idea is that the same keys will have the same hash, thus partitioning them together makes most sense. With a quick google search, it seems that SPARK uses hashpartitioner as the default partitioner if none is specified. However, there is also RangePartitioner and CustomPartitioner.
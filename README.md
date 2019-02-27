##Assignment 2##
Here is my blog post on assignment 2. Assignment 2 was about the Map-Reduce framework and how to write code that can run on an hadoop cluster. In the lectures I got to learn how map-reduce code works and how to write working code. Here is what I did:

1. First, I started a docker container and ran the hadoop file system in it. 
2. Then, I downloaded the 'Complete Shakespeare' file in docker and also put a copy of it in the hadoop file system.
3. With the editor vim, I created a file WordCount.java taken from the [Map-Reduce documentation: ](https://hadoop.apache.org/docs/r2.7.3/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0)
4. Next, I ran the code on the 'Complete Shakespeare' file in the hadoop file system and saved the output in a file called 'output'. The code from WordCount.java saves all words and how many times they occur in 'Complete Shakespeare'.

Because hadoop is a different file system, in order to get the outputs of the jobs that I ran I had to:

- copy the files from hdfs to local, which was docker, and then 
- copy the files from docker to the local file system. 

Running the command 'hdfs dfs -cat output/part-r-00000' would display the outputs of the job on in the command prompt but I wanted to open the files from home. 

Below is the pseudocode for the mapper in WordCount.java:

	public void map(key,value){  
	    for word in value:  
	        emit(word,1)  
	}

And the reducer:

	public void reduce(key,values){  
		sum=0  
		for val : values:  
			sum+=val` 
		}
		emit(key,result)  
	}

During the mapper phase, words are emitted after every word with a value of 1. During the reducer phase, every word will have a list of values which are summed up and emitted together with the word. An example from the output :

	'The	29
	'Then	3
	'Then,	2
	'There	1
	'There's	1
	'These	1
The Map-Reduce code does not take into account special characters. If we want to get the words so that special characters are filtered out, we need to do this in the map phase by using a regular expression to filter special characters out and emit the filtered word. 

To get the total word count (of all words), the mapper can be changed to emit the same word every time. For example:

	public void map(key,value){
		for word in value:
			emit("",1)
	}

There are 959301 words in total in the file.

Counting the number of lines is easy. Simply emitting the same word (perhaps an empty word) and the value of 1 does it. There are 147838 lines in the 'Complete Shakespeare' file. It seems like every mapper gets 1 line as input. Below is pseudocode for counting the number of lines:

	public void map(key,value){  
	    emit("",1)  
	}

To count the number of characters, change the mapper loop over every word and emit each character. Example:

	public void map(key,value){  
	    for word in value: 
			for c in word: 
	        	emit(c,1)  
	}
Example output: 

	a	263741
	b	50577
	c	72571
	d	145094
	e	442637
	f	74636
	g	62204
	h	238451
	i	216326

Now we answer the question: Are there more Romeo or Juliet occurrances in the 'Complete Shakespeare' file? To do that, the mapper has to be changed so that it emits only when the word contains either 'Romeo' or 'Juliet'. An example output:

	'Juliet.']	1
	'Romeo	2
	JULIET	4
	JULIET,	2
	JULIET.	125
	JULIET]	1
	Juliet	17

I immediately noticed that the mapper did not take into account special characters. So, after filtering out special characters and ignoring upper cases, here is the output:

	Juliet	206
	Romeo	313

As we can see, Romeo occurs more times than Juliet.
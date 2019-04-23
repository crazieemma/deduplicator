# README
# I. Documentation
### **Description**
 Deduplicator Project is seeking to implement efficiently store Data in Lockers. This implementation need Compress when store data and Decompress when retrieve data.
### **Group member**
+ Yuying Wang(yuywang@bu.edu)
+ Yuxuan Liu(emmaliu@bu.edu)
+ Yueling Jiang(jyueling@bu.edu)
+ Sichun Hao(sichunh@bu.edu)
+ Tianyi Sun(tysun@bu.edu)
### **Design decisions**
+ **Algorithm**
    + [MD5](https://en.wikipedia.org/wiki/MD5)
        + To check if the two strings are same 
    + [Sunday Algorithm](https://csclub.uwaterloo.ca/~pbarfuss/p132-sunday.pdf)
        + In the matching process, the pattern string is matched one by one from left to right at first. When a mismatched character is found, the current matching position skips the step size as large as possible, and then the next round of matching is carried out, so as to improve the matching efficiency. When the Sunday algorithm fails to match, it determines whether the next character of the last character in the main string participating in the match appears in the pattern string. If not, the jump step length is the length of the pattern string plus 1; If present, the jump step is the distance from the rightmost end of the pattern string to the end of the character plus 1. 
        + Assume the compared file has n characters, the amortized time for Sunday Algorithm is **O(n)**.
    + [Long Common Substring Using Dynamic Programming](https://en.wikipedia.org/wiki/Longest_common_substring_problem)
        + A longest substring is a sequence that appears in the same order and necessarily contiguous in both the strings.
        + Create a matrix of size of m*n and store the solutions of substrings to use them later.Using [bottom-up](https://algorithms.tutorialhorizon.com//introduction-to-dynamic-programming-fibonacci-series/#Bottom-Up) manner.
        + Base Cases: If any of the string is null then LCS will be 0.Check if ith character in one string A is equal to jth character in string B
            + Case 1: both characters are same
LCS[i][j] = 1 + LCS[i-1][j-1] (add 1 to the result and remove the last character from both the strings and check the result for the smaller string.)
            + Case 2: both characters are not same.
LCS[i][j] = 0. At the end, traverse the matrix and find the maximum element in it, This will the length of Longest Common Substring.
        + Running time: O(m*n) time
+ **Attention**
    + For the requirments from the project description, We only support ASCII enconding. If you use files which are not content to it, we will convert them into ASCII which would cause some gibberishes in your file.
+ **Explaination of some file**
    + common.txt : contains the indexes(targetfile,reference,length)
    + reference.txt : the reference for laer
    + file.txt : the difference chars from the reference.txt saved in this file
### **Features**
#### 1) Basic feature
+ **Ability to store fifty 10MB ASCII files, using at most 100MB of storage.**
    + divideFactor: depend on the length of file contents
        + split the whole file to “divideFactor” parts in order to run faster and get more refined deduplication. 
    + For each part of the text, we are going to do separate dedup operation.
        + Firstly, use MD5 to compress the text, if results of two files are the same, then store nothing in file2, store the compare index of start and length in common.txt..
        + If the results are not the same, then use Sunday algorithm to found the max-length-same-substring and return its begin indexes in both files and the length of the substring.  
        + Because the Sunday Algorithm has some limits, so if we can not get the result from the Sunday, we will use the Dynamic Algorithm to find the long common substring.
    + In the output, we store a reference file, a deduped file2 and a .txt store all the index of same characters. 
+ **Ability to retrieve all files stored in the locker in any order, at any time.**
    + *The retrieve function basically cal decompress function. The decompress has two constructors and you can call it entering locker as well as file name or enter them externally. The first step of retrieve, is to check wether this file is image or text file. If it is an image file, it will decompress and store the retrieved image string text in locker. Then, decompresser will read three data files to perform decompressing — inference file, index file, and compressed file. For each line of index file, according to these three column data, it read part of compressed file as first part, find certain substring of reference, and read following part of compressed file, combing this three part together to form a new temporary string. Based on this string, we used next line of index file to perform the same operation, until the last line of index file. The whole string will be return as the decompressed file.*
+ **Your locker should be portable**
    + Locker is stored as a folder in computer and can be transfered easily
+ **Command-line User Interface**
    + we support add plain text, image and store one directory as one entity
        +   For adding file to the locker
        `java dedup -addFile [file] -locker [locker name]`
            + tips: [file] means the absolute path of the file
        + For retrieve file from the locker
        `java dedup -retrieveFile [file] -locker [locker name]`
        + For delete file from the locker
        `java  dedup -deleteFile [file] -locker [locker name]`
        + For search substring from the locker
        `java dedup -search [string] -locker [locker name]`
            + tips:  [string] means the substring you want to search, you need to use "" when your substring has space
        + For display storage
        `java dedup -display storage -locker [locker name]`
            + tips: you can only change the [locker name]


#### 2) Possible feature
+ **Develop networked access to your locker.**
    + *We use java socket communication to achieve networked access to our locker. After TCP three way handshakes, connection between Server and Client is built. Client could send locker name and file to upload file; could send locker name and file name to retrieve or delete file; also could send string to search if it is in some file in the locker. After doing those functions, Server would send feedback to client (i.e. retrieved file, storage progress…).*
+ **Ability to efficiently store similar images.**
    + *Convert the image to base64 String using Base64.encodeBase64, after converting, use the same method as .txt file to do the compressor for store and decompressor for retrieve.*
+ **Develop a Graphical User Interface that displays storage progress.**
    + *Using java Swing framework to implement basic Gui layout, including label, button, text field, dialog input panel and option panel. By clicking ‘choose file’ button,  you can choose file from local directory and the file path will show on the right. Remember to enter locker name you want. After clicking submit, it will deduplicate your file and store in the locker. To retrieve a file, enter the filename as well as locker name, click ‘retrieve’ button and check your retrieved file in RetrievedFiles folder. Also, you can delete file in a certain locker, by entering filename and clicking delete. By entering a string, click search button and it will show the result of file name that contains this string. Successful or failed operation will get message dialog to tell you the result. I add action listener class to each button, that in the way, it can connect with Client and have interface with Server.*
+ **Ability to store directories of files as one entity.**
    + *After user input a folder, it traverses all the files use recursion and store each file’s relative directories (to root folder) in an ArrayList in order to record the subfolders. In this way, the folder path will also appear in the ArrayList. Then, we try to see if this file is text file or image file, and store them one by one into the locker after compression.*
+ **Implement file deletion from your locker.**
    + *After user input a filename and locker name to delete the file, we need to delete the file “filename.txt” and “filename_common.txt” in the locker. If input filename does not exist, we return null. If both files are deleted, it is deleted success. Otherwise, delete fail and it returns null.*
+ **implement efficient substring search.**
   + *Find the Long common substring of substring and reference.txt, extend the indexes of the LCS to a larger range.(e.g [1000,1200] to [1000-substring.length, 1200+substring.length]). Using the decompressor to decompress the extended part(traverse all common.txt and corresponding file.txt), check if the substring is contained in the decompressed string, if true, add the filename to the result.*
+ **Develop an Android client for your locker.**
    + We build a “Basic Activity” then mainly changed 5 files in this project. To develop an android client, we need to build two parts, one is the layout of GUI, the other is to specified the function of each buttons.
    + Declare Button name and connect it to specified id
    + Use switch case to change the function of each button and plant text
        + When press “load and compress”, we load file from “raw”, the resource folder. After that, call “Compressor.java” to generate the deduplicated files. The content will be stored in a String then put in a file in SDcard (the external storage used as locker) byte by byte.
        + When press “decompress”, we call “decompressor.java” to use the file we stored in SDcard to retrieve the files. 
        + When click on “delete” button, the code will check whether the file exist, then delete it if it has been found.
    + We also set some place of plaintext in order to show the file we store, dedup and retrieve. In Android Studio, the SDcard is an .img file witch cannot be open directly, so we just show them on screen.
    + For more specific information, clink [here](https://drive.google.com/file/d/1JBCe6Fcso74E5HZC9PMWFxZh3uq54koB/view)

            
   
### **Reference and backgroud material**

+ Sunday Algorithm to find the same substring:  https://csclub.uwaterloo.ca/~pbarfuss/p132-sunday.pdf
+ Extreme Binning: Scalable, Parallel Deduplication for Chunk-based File Backup https://www.computer.org/csdl/proceedings/mascots/2009/4927/00/05366623.pdf
+ Socket: https://docs.oracle.com/javase/7/docs/api/java/net/Socket.html
+ Frequency Based Chunking for Data De-Duplication https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5581583
+ Data deduplication techniques https://ieeexplore.ieee.org/abstract/document/5656539
+ Awt: https://docs.oracle.com/javase/7/docs/api/java/awt/event/ActionEvent.html
+ LCS using Dynamic Program: https://en.wikipedia.org/wiki/Longest_common_substring_problem



# II. Code
### **File name:**
+  CommondLine
    + **Zip.java** : one entity
    + **search.java** : search substring
    + **sundayAlgo.java** : algorithm for compressor
    + **DeleteFile.java** : file deletion
    + **Compressor.java** : compress file
    + **Decompressor.java** : decompress file
    + **similarImage.java** : store similar image 
+ Network
    + **server.java**
    + **client.java**
+ GUI
    + **GUI.java**
    + **dedupWeb.jar**
+ Android
    + **MyApplication_v2.zip**
+ Test
    + **unit test** : all the files in the unit_test folder
    + **System test** 
        + Use the commands in the 'Command-line User Interface' to test our code
        + Use the GUI to test


### **Data:**
+ **textFiles folder**: including the original files, for test **compressor.java** and **zip.java**
+ **testlocker folder**: including all compressed files, for test **search.java**, **decompressor.java** and **DeleteFile.java**
+ **testImg folder**: including similar images, for test **similarImage.java** and **commondline.txt**
# III. Work breakdown
+ **Yuying Wang:** 
    + dedup
    + unit test
    + Commandline
    + Compressor Algorithm accompany with Sichun Hao and Yuxuan Liu
+ **Yuxuan Liu:**
    + unit test 
    + Substring search
    + Similar image storage
    + Compressor Algorithm accompany with Sichun Hao and Yuying Wang
+ **Yueling Jiang:** 
    + unit test    
    + Save as one entity 
    + Decompressor accompany with Tianyi Sun
    + Network access to locker accompany with Tianyi Sun
+ **Sichun Hao:**
    + unit test
    + File deletion
    + Android client
    + Compressor Algorithm accompany with Yuxuan Liu and Yuying Wang
+ **Tianyi Sun:**
    + unit test
    + GUI implementation
    + Decompressor accompany with Yueling Jiang
    + Network access to locker accompany with Yueling Jiang

# IV. Jira links
This is our [ Jira Link](https://agile.bu.edu/jira/projects/GROUP11/issues/GROUP11-10?filter=allopenissues).



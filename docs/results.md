#Inception结果集

Inception给用户返回的信息有两种，一种是提交给Inception的基础信息存在错误，比如源信息不全，或者源信息有错误等，这种情况下，直接报异常，包括错误码及错误信息，与MySQL服务器的异常是一样的，在外面正常处理即可。
如果没有上面的问题，都会以结果集的方式将检查结果告诉客户端。这种方式非常友好，在它的基础上用现有的成熟的接口处理起来非常方便。返回的结果集中，每一个行数据，就是一条提交的SQL语句，Inception内部将所有提交的语句块一条条的拆开，以结果集的方式返回，针对每一条语句，有什么问题或者状态，在结果集中是一目了然。

**需要注意的是**，如果在语句中出现语法错误，则不能继续了，因为Inception已经不能将剩下的语句分开了，那么此时前面已经正常检查的多行为多个结果集的行返回，后面出错的语句为一行返回，当然这个的错误信息是语法错误。
Inception返回结果集的结构如下：  

1. **ID**：用来表示检查的sql序号的，每次检查都是从1开始。    
2. **stage**：这个列显示当前语句已经进行到哪一步了，包括CHECKED、EXECUTED、RERUN、NONE，NONE表示没有做过任何处理，有可能前面有语法错误直接就提前返回了, CHECKED表示这个语句只做过审核，而没有再进行下一步操作，EXECUTED表示已经执行过，如果执行失败，也是用这个状态表示，RERUN表示的是，对于影响上下文的语句，已经执行成功，但为了与EXECUTED区分，用RERUN表示，主要是因为在执行过程中，如果某一条语句执行失败了，则上层可能需要将没有执行的语句提取出来，再次执行，那么影响上下文的语句是需要加上的，所以用RERUN来表示。影响上下文的语句一般包括set names和use db这两种，而当前Inception支持的只有这两种。
3. **errlevel**：返回值为非0的情况下，说明是有错的。1表示警告，不影响执行，2表示严重错误，必须修改。  
4. **stagestatus**：用来表示检查及执行的过程是成功还是失败，如果审核成功，则返回 Audit completed。如果执行成功则返回Execute Successfully，否则返回Execute failed，如果备份成功，则在后面追加Backup successfully，否则追加Backup failed，这个列的返回信息是为了将结果集直接输出而设置的，如果在具体使用过程中，为了更友好的显示，可以在这基础上再做加工处理。  
5. **errormessage**：用来表示出错错误信息，这里包括一条语句中所有的错误信息，用换行符分隔，但有时候如果某一个错误导致不能继续分析了，则后面的错误就不能显示出来。如果没有出错，则用显示为None。而对于执行及备份错误，因为对于一条语句，这样的错误只会有一次，那么执行错误会在后面追加“execute:具体的执行错误原因”，如果是备份出错，则在后面追加“backup:具体的备份错误原因”，而在执行时，有时候会出现Warnings，比如插入数据时字符串被截断啥的，此时会输出这些warnings：`#1 Execute(Warning, Code errno):warning message`，#号后面的数字表示第几个警告，因为有时候执行一个语句会产生多个警告。  
6. **SQL**：用来表示当前检查的是哪条sql语句。如果某一条sql语句在检查时有语法错误，则这里面会包括从出错语句开始到后面所有的语句，因为语法出错后实在是真的不能再继续分析了，也就不能将后面的每条语句分开了，这个列还会有一个特别的地方，如果当前语句是`inception show xxxx`命令集中的第一种情况（<<**Inception命令集**>>中会做具体介绍）。  
7. **affected_rows**：用来表示当前语句执行时预计影响的行数，在执行时显示的是真实影响行数。  
8. **sequence**：这个列与上面说的备份功能有关系，其实就是对应**$_$Inception_backup_information$_$**表中的 opid_time 这个列，一一对应，这就为前端应用在针对某一操作回滚找到了入口，每次执行都会产生一个序号，如果要回滚，则就使用这个值从备份表中找到对应的回滚语句执行即可。  
9. **backup_dbname**：这个列表示的是当前语句产生的备份信息，存储在备份服务器的哪个数据库中，这是一个字符串类型的值，只针对需要备份的语句，数据库名由IP地址、端口、源数据库名组成，由下划线连接，而如果是不需要备份的语句，则返回字符串None。  
10. **execute_time**：这个列表示当前语句执行时间，单位为秒，精确到小数点后两位。列类型为字符串，使用时可能需要转换成DOUBLE类型的值，如果只是审核而不执行，则这个列返回的值为0。  

#友情提示
上面的列，是为了尽可能的丰富的，更灵活的让上层使用，所以列比较多，而如果在具体使用中，如果哪些列觉得没什么意义或者用处不大，可以不关心，特别是只做一个审核页面的话，那么很多列都是不需要关心的，只需要关心ID、errlevel、errormessage、SQL及affected_rows即可。总之一句话，灵活应用即可。当然有任何问题或者疑问都可以随时联系本人（见首页）。

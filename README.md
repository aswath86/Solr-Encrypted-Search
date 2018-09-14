# Solr-Encrypted-Search
Index encrypted files, enable search on them, and not expose the data in Search


# Welcome to the Solr-Encrypted-Search wiki!

### Use Case: How to enable search on a file/data after it is encrypted before it is stored in the search index. And also make the data not-visible in the search 

Let's assume we use Solr as as our Search Engine and we index 3 records into it,

`{`
	`"id": 1,`
	`"content": "This is a sample title with no data"`
`}, {`
	`"id": 2,`
	`"content": "I need to ecrypt my search data for privacy"`
`}, {`
	`"id": 3,`
	`"content": "This data is protected"`
`}`

But before we index let's do the schema like this,

`<field name="content" type="text_general_encrypt" indexed="true" stored="false" multiValued="false"/>`

` <fieldType name="text_general_encrypt" class="solr.TextField" positionIncrementGap="100">`
      `<analyzer type="index">`
        `<tokenizer class="solr.KeywordTokenizerFactory"/>`
      `</analyzer>`
      `<analyzer type="query">`
        `<tokenizer class="solr.KeywordTokenizerFactory"/>`
      `</analyzer>`
    `</fieldType>`

For the sake of simplicity let's use the encryption logic from this page, [https://howtodoinjava.com/security/java-aes-encryption-example/](https://howtodoinjava.com/security/java-aes-encryption-example/)
Credits to @Lokesh Gupta 

The code used is below,


	public static void main(String[] args) {
		final String secretKey = "ssshhhhhhhhhhh!!!!";

		String[] originalStringAry = "This is a sample title with no data".split(" ");

		String finalEncryptedString = "";

		for (int i = 0; i < originalStringAry.length; i++) {
			String encryptedString = AES.encrypt(originalStringAry[i], secretKey);
			//String decryptedString = AES.decrypt(encryptedString, secretKey);
			finalEncryptedString = finalEncryptedString + " " + encryptedString;

		}
		System.out.println(finalEncryptedString);

	}

Note: Let's focus on the problem statement at hand and not on the code or the coding standard or the encryption algorithm. The encryption algorithm can be changed/could be done better.

We get the following encrypted data,

`{`
	`"id": 1,`
	`"content": "L0C7QTRS+DShuAQ+owHI9g== jKduOo6HLWyaohywGqnK2g== dO4qW/9cX22gM67c/NTzrg== TQBuDsJpDAb0eYEBpeS0SA== 978GMKHzXap4iPJujJ7bow== 850+JgxY8iLZYYTSI08+Jg== b2eO+gpXAgbYHVtFpj4Npw== fwUa0PyoTeDl/AUidFlWcg=="`
`}, {`
	`"id": 2,`
	`"content": "R+DEERdCKW+MIK8hV1TWIw== 8uRZC9TIgqtJLl49Mgwh1Q== /Nm2r/jIFssyPn2VH5KAZg== n4dfNBCl9g6zsWoDjFIu4w== hwrfYyRqhHolpHD0hL1Xcw== EwN5LpsX27ie4upqX7BAiA== fwUa0PyoTeDl/AUidFlWcg== hei8EywBg7V31OFOjWjPnA== hl/fvllCTr/sTv/gK1Vm6w=="`
`}, {`
	`"id": 3,`
	`"content": "L0C7QTRS+DShuAQ+owHI9g== fwUa0PyoTeDl/AUidFlWcg== jKduOo6HLWyaohywGqnK2g== 1ZHsI7OCsPX9bqfkdXOQ7A=="`
`}`

Index them like this,

![](https://github.com/Aswath86/Solr-Encrypted-Search/blob/master/Solr_Indexing.png)

Now do a search for *

![](https://github.com/Aswath86/Solr-Encrypted-Search/blob/master/Solr_Search_All.png)

Now let's do a search for "sample" (ofcourse, without double quotes). But before we send the search text to Solr, we need to encrypt the data and send it. And when we encrypt the search term we get below (ofcourse, without double quotes),

Note: The front-end search application has to encrypt the search term before it is passed to the search engine as a query. 

"TQBuDsJpDAb0eYEBpeS0SA=="

Note: Need to make sure we use the same encryption/decryption logic

When we do a search for "sample" we are suppose retrieve record 1 and nothing else. Let's see,

![](https://github.com/Aswath86/Solr-Encrypted-Search/blob/master/Search1.png)

When we do a search for "data" we are suppose retrieve record 1, 2 & 3. Let's see,

![](https://github.com/Aswath86/Solr-Encrypted-Search/blob/master/Search2.png)

When we do a search for "ecrypt OR protected" we are suppose retrieve record 2 & 3. This is a boolean search. Let's see,

![](https://github.com/Aswath86/Solr-Encrypted-Search/blob/master/Search3.png)

Bingo!

However, ideally the encryption and decryption logic has to be in-built with search. It's quite easy to build a custom filter in Solr and Elasticsearch and to deploy & use them. Please refer below article,

[https://dzone.com/articles/build-a-custom-solr-filter-to-handle-unit-conversions](https://dzone.com/articles/build-a-custom-solr-filter-to-handle-unit-conversions)
Credits to @Sumeet Sharma

Something like this,

`   <fieldType name="text_general_encrypt" class="solr.TextField" positionIncrementGap="100">`
      `<analyzer type="index">`
        `<tokenizer class="solr.KeywordTokenizerFactory"/>`
		`<filter class="com.solr.custom.filter.test.CustomEncryptionTokenFilterFactoryIndexer" />`
      `</analyzer>`
      `<analyzer type="query">`
        `<tokenizer class="solr.KeywordTokenizerFactory"/>`
		`<filter class="com.solr.custom.filter.test.CustomEncryptionTokenFilterFactoryQuery" />`
      `</analyzer>`
    `</fieldType>`

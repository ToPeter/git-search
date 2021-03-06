# git-search

Simple CLI aimed at 'usage research' backed by github.
This may one day become part of bigger 'business intelligence' tools, to support

- Architects decision making (what to use)
- Best practices searches (how to use it, how others use it ?)
- Bad practices, QA tools (wrong usage, security vulnerabilities, bad performance codes)

## Mission

It is often needed to find out 'how many projects (repos, people) are using this framework, this api etc...

It seems that the results achievable by using gitgub search API are not satisfactory (see problems section),
so we implement this mini project as workaround.

## Stories

- gulp vs. grunt
- test frameworks used in node.js projects (mocha, vs. tap)
- ...


### gulp vs. gunt

To find 'repos using grunt' run the following command:	
	
	TODO: command line

To find 'repos using gulp' run the following command:

	TODO: command line

## Problems

Just short summary, read below for more details:

- repo search - limited 'in' and query capabilities
- code search API - Must include at least one user, organization, or repository

## Background Story

Repos Using Grunt = repos with Gruntfile.js. 
1) How many Gruntfile.js are on github ? 
2) Repository names having ./Gruntfile.js 

[Github API][Github-API] provides options to search:

- Search repositories
- Search code
- Search issues
- Search users
- Text match metadata

Search Repositories (NOT USABLE)

It seems that [searching repositories][searching-repositories] is NOT USABLE for our purpose, 
it can only search in name, description and readme. 

Search code (NOT USABLE)

Looking at [searching-code][searching-code] this seems to be an option:

```Gruntfile.js in:filename language:JavaScript```

<https://github.com/search?l=javascript&q=Gruntfile.js+in%3Afilename+language:JavaScript&ref=searchresults&type=Code&utf8=✓>

However when running with API 

	curl 'https://api.github.com/search/code?q=Gruntfile.js+in:filename+language:JavaScript'
	
its responds with:

	{
	  "message": "Validation Failed",
	  "errors": [
	    {
	      "message": "Must include at least one user, organization, or repository",
	      "resource": "Search",
	      "field": "q",
	      "code": "invalid"
	    }
	  ],
	  "documentation_url": "https://developer.github.com/v3/search/#search-code"
	}

This is documented limitation (TODO: link).

## Web Scraping
When API fail, we have to resolve to [Web Scraping][Web-Scraping].

Sniffing Github UI we have discovered that two URS exists:

Calling:
	
	https://github.com/search?l=javascript&q=Gruntfile.js+in%3Afilename+language:JavaScript&ref=searchresults&type=Code&utf8=✓

Will result in 'these services' called:
	
	https://github.com/search/count?l=javascript&q=Gruntfile.js+in%3Afilename+language%3AJavaScript&ref=searchresults&type=Issues&utf8=%E2%9C%93
	https://github.com/search/count?l=javascript&q=Gruntfile.js+in%3Afilename+language%3AJavaScript&ref=searchresults&type=Repositories&utf8=%E2%9C%93
	https://github.com/search/count?l=javascript&q=Gruntfile.js+in%3Afilename+language%3AJavaScript&ref=searchresults&type=Users&utf8=%E2%9C%93
	
	https://github.com/search?l=javascript&q=Gruntfile.js+in%3Afilename+language:JavaScript&ref=searchresults&type=Code&utf8=%E2%9C%93

It is calling 3 counts and one 'detailed search results'.

To get count of Code:

	
	curl 'https://github.com/search/count?l=javascript&q=Gruntfile.js+in%3Afilename+language%3AJavaScript&ref=searchresults&type=Code&utf8=%E2%9C%93'
	
	<span class="counter">257,639</span>

To get details of Code:

	curl https://github.com/search?l=javascript&q=Gruntfile.js+in%3Afilename+language:JavaScript&ref=searchresults&type=Code&utf8=%E2%9C%93

However this returns 'FALSE POSITIVES':

	Example:
	sylvinus/backbone-simpleapp-kitlers – grunt.js 
	
	
- Can we make the query more specific ? No.
The repository search feature doesn't allow a filter which matches based on files included in repositories
 
Workaround:
Do it via the Search API

https://developer.github.com/v3/search/#search-code

Each result will have a repository object which tells us which repository the file is in. However, the code Search API doesn't allow you to run global searches -- we need to scope our search to a specific file or repository:

https://developer.github.com/changes/2013-10-18-new-code-search-requirements/

Also, we can fetch up to 1000 results:

https://developer.github.com/v3/search/#about-the-search-api

That means that we would need to collect the data incrementally. For example, we could first fetch a list of all JavaScript or CoffeeScript repositories via the Repositories search API (https://developer.github.com/v3/search/#search-repositories) and then search for grunt.js files within those repositories 1-by-1 using the code search API.

- if no we need to loop all results and filter out manually false positives
- other ideas ?	


node.js tools:

- <https://www.npmjs.com/package/htmlparser2>
- <https://www.npmjs.com/package/cheerio-cli>

TODO: continue.



<!-- reference style links -->

[Github-API]: https://developer.github.com/v3/search/ 
[searching-repositories]: https://help.github.com/articles/searching-repositories/
[searching-code]: https://help.github.com/articles/searching-code/
[Web-Scraping]: https://en.wikipedia.org/wiki/

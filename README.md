# hello-world
# -*- coding: utf-8 -*-
import time
from html.parser import HTMLParser
import numpy as np
import urllib
from urllib import request
from openpyxl import Workbook
hds=[{'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.6) Gecko/20091201 Firefox/3.5.6'},{'Mozilla/5.0 (Windows NT 6.2) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.12 Safari/535.11'},{'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Trident/6.0)'}]
def do_spider(book_tag):
	book_lists=[]
	page_num=0
	total_len=[]
	len_tag=0
	for page_num in range(15):
		# url='https://www.douban.com/tag/%E5%B0%8F%E8%AF%B4/book?start=0'
		url='https://www.douban.com/tag/'+urllib.parse.quote(book_tag)+'/book?start='+str(page_num*15)
		reg=request.Request(url)
		reg.add_header('User-Agent',hds[page_num%len(hds)])
		with request.urlopen(url) as f:
			source_code=f.read().decode('utf-8')
		time.sleep(np.random.rand()*5)
		html1=source_code
		class MyHTMLParser(HTMLParser):  #解析html数据
			enter_a=False
			enter_desc=False
			enter_rating=False
			def handle_starttag(self,tag,attrs):
				if tag=='a':
					for (variable, value) in attrs:
						if value=='title':
							self.enter_a=True
				if tag=='div':
					for (variable, value) in attrs:
						if value=='desc':
							self.enter_desc=True
				if tag=='span':
					for (variable, value) in attrs:
						if value=='rating_nums':
							self.enter_rating=True
			def handle_endtag(self,tag):
				if tag=='a' and self.enter_a:
					self.enter_a=False
				if tag=='div' and self.enter_desc:
					self.enter_desc=False
				if tag=='span' and self.enter_rating:
					self.enter_rating=False
			def handle_data(self,data):
				global book_list
				if self.enter_a:
					book_list={}
					book_list['title']=data
				if self.enter_desc:
					book_list['desc']=data.strip()
				if self.enter_rating:
					book_list['rating']=data
					# global book_lists
					book_lists.append(book_list)
		parser=MyHTMLParser()
		parser.feed(html1)
		if len_tag != len(book_lists):
			len_tag=len(book_lists)
		else:
			break
		# total_len.append(len(book_lists))
		# for i in range()
		# print(total_len)
		print('Downloading Information From tag %s,pag %d,TotalNum %d'%(book_tag,page_num,len(book_lists)))
	return book_lists
def book_spider(book_tag_list):
	books=[]
	for book_tag in book_tag_list:
		book_lists=do_spider(book_tag)
		books.append(book_lists)
	# print(books[0])
	# print(books[1])
	return books
def print_book_lists_excel(books,book_tag_lists):
	wb = Workbook(write_only=True)
	ws=[]
	for j in range(len(book_tag_lists)):
		ws = wb.create_sheet(title=book_tag_lists[j])
		ws.append(['序号','书名','评分','信息'])
		count=1
		for i in books[j]:  #按标签把爬下的数据写入
			ws.append([count,i['title'],float(i['rating']),i['desc']])
			count+=1
	save_path='book_list'
	save_path+='.xlsx'
	wb.save(save_path)
book_tag_list=['小说','科幻','爱情']
books=book_spider(book_tag_list)
print_book_lists_excel(books,book_tag_list)






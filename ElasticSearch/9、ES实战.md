# ES实战

## 爬取数据入库

使用java Jsoup解析Html页面，爬取数据后解析为对象，然后存入es中

```xml
 <!--网页解析-->
    <dependency>
      <groupId>org.jsoup</groupId>
      <artifactId>jsoup</artifactId>
      <version>1.12.1</version>
    </dependency>
```

爬取代码

```java
private static final String URL= "https://search.jd.com/Search?keyword=";

    public List<Book> get( String keyword, Integer page) throws Exception {
        List<Book> result = new ArrayList<>();
        for (int i = 1; i <= page; i++) {
            String curUrl = URL + keyword+"&page="+i;
            Document document = Jsoup.parse(new URL(curUrl), 20000);
            Element jGoodsList = document.getElementById("J_goodsList");
            //System.out.println(jGoodsList.html());
            Elements li = jGoodsList.getElementsByTag("li");
            for (Element element : li) {
                String img = element.getElementsByTag("img").eq(0).attr("src");
                String price = element.getElementsByClass("p-price").eq(0).text();
                String title = element.getElementsByClass("p-name").eq(0).text().split(" ")[0];
                String shop = element.getElementsByClass("curr-shop hd-shopname").eq(0).text();
                //System.out.println("=================================");
                if(!StringUtils.isEmpty(img)){
                    Book book = new Book();
                    book.setId(UUID.randomUUID().toString());
                    book.setPrice(price);
                    book.setImg(img);
                    book.setTitle(title);
                    book.setShop(shop);
                    result.add(book);
                }
            }
        }
        return result;
    }
```

## 存入es

```java
@Autowired
    private HTMLParseUtil htmlParseUtil;

    @Autowired
    private BookService bookService;

    @GetMapping("/init/{keyword}/{page}")
    private Object initBook(@PathVariable(value = "keyword") String keyword,@PathVariable(value = "page") Integer page) throws Exception {
        List<Book> books = htmlParseUtil.get(keyword, page);
        log.info(String.valueOf(books));
        try {
            bookService.saveAll(books);
        } catch (Exception e) {
            log.error("init fail");
            return "init fail";
        }
        return "init success";
    }
```

## 条件查询


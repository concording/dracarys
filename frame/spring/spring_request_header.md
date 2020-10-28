

```
org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported
```

The problem is that when we use application/x-www-form-urlencoded, Spring doesn't understand it as a RequestBody. 
So, if we want to use this we must remove the @RequestBody annotation.

```
@RequestMapping(value = "/order", method = RequestMethod.POST,
            consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE,
            produces = {MediaType.APPLICATION_ATOM_XML_VALUE, MediaType.APPLICATION_JSON_VALUE})
    @ResponseBody
    String index(@RequestParam Map<String, String> paramMap) throws Exception{
    }
```

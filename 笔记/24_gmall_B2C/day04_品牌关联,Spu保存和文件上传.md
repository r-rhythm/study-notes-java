# 品牌列表接口
## 完成品牌分页查询接口
`/admin/product/baseTrademark/{page}/{limit}`
单表的操作可以直接集成在 controller,service专注业务即可
操作 base_trademark 品牌表完成

## 根据三级分类查询对应的品牌集合
使用stream流做过滤
```Java
/**  
 * 根据第三级id查询出对应的品牌  
 *  
 * @param category3Id 三级分类id  
 * @return 品牌List集合  
 */  
@Override  
public List<BaseTrademark> getTrademarkListByCategory3Id(Long category3Id) {  
      
    LambdaQueryWrapper<BaseCategoryTrademark> wrapper = new LambdaQueryWrapper<>();  
    wrapper.eq(BaseCategoryTrademark::getCategory3Id, category3Id);  
      
    List<BaseCategoryTrademark> baseCategoryTrademarks = baseCategoryTrademarkMapper.selectList(wrapper);  
      
    if (baseCategoryTrademarks == null) {  
        return Collections.emptyList();  
    }  
      
    // 在中间表中通过 category3Id 查询出对应的多个对象,获取他们的 trademarkId    
    List<Long> trademarkIds = baseCategoryTrademarks  
            .stream()  
            .map(BaseCategoryTrademark::getTrademarkId)  
            .collect(Collectors.toList());  
      
    // 如果没有对应的id,直接返回空集合  
    if (CollectionUtils.isEmpty(trademarkIds)) {  
        return Collections.emptyList();  
    }  
      
    // 使用 tradeId 查询出对应的品牌名称并返回  
    return baseTradeMarkMapper.selectBatchIds(trademarkIds);  
}
```

## 删除分类品牌关联

## 展现当前分类下没有关联的品牌信息(可选品牌集合)
如果当前分类下没有关联的品牌,就给他返回当前分类下的所有品牌
```Java
/**  
 * 根据三级id查询还未被关联的品牌  
 *  
 * @param category3Id 三级id  
 * @return  
 */  
@Override  
public List<BaseTrademark> findCurrentTrademarkList(Long category3Id) {  
    LambdaQueryWrapper<BaseCategoryTrademark> wrapper = new LambdaQueryWrapper<>();  
    wrapper.eq(BaseCategoryTrademark::getCategory3Id, category3Id);  
      
    // 根据三级id查询出目前已经关联的品牌  
    List<BaseCategoryTrademark> baseCategoryTrademarks = baseCategoryTrademarkMapper.selectList(wrapper);  
    if (!CollectionUtils.isEmpty(baseCategoryTrademarks)) {  
        List<Long> trademarkIds = baseCategoryTrademarks.stream()  
                .map(BaseCategoryTrademark::getTrademarkId)  
                .collect(Collectors.toList());  
          
        // 查询所有品牌,将当前没有关联的品牌查询筛选出来并返回  
        return baseTradeMarkMapper.selectList(null)  
                .stream()  
                .filter(baseTrademark -> !trademarkIds.contains(baseTrademark.getId()))  
                .collect(Collectors.toList());  
    }  
    // 如果当前的三级id没有关联任何的品牌,给他返回所有的品牌  
    return baseTradeMarkMapper.selectList(null);  
}
```

# SPU 保存功能
## 图片上传
使用[[对象存储服务#MinIo]] 完成图片上传
```Java
/**  
 * @author: liu-wēi  
 * @date: 2022/12/1,13:33  
 */@RestController  
@RequestMapping("/admin/product")  
public class FileUploadController {  
      
    /**  
     * 从配置中心获取minio配置  
     */  
    @Value("${minio.endpointUrl}")  
    public String endpointUrl;  
    @Value("${minio.accessKey}")  
    public String accessKey;  
    @Value("${minio.secreKey}")  
    public String secreKey;  
    @Value("${minio.bucketName}")  
    public String bucketName;  
      
      
    /**  
     * 文件上传接口  
     *  
     * @param file  
     * @return  
     */  
    @PostMapping("/fileUpload")  
    public Result fileUpload(MultipartFile file) {  
        String fileUrl = "";  
        try {  
            MinioClient minioClient = MinioClient  
                    .builder()  
                    .endpoint(endpointUrl)  
                    .credentials(accessKey, secreKey)  
                    .build();  
            boolean exists = minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());  
            if (exists) {  
                System.out.println("Bucket already exists.");  
            } else {  
                // 如果不存在,则创建桶  
                minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());  
            }  
              
            // 文件名称  
            String fileName = UUID.randomUUID().toString().replaceAll("-", "") + file.getOriginalFilename();  
              
            // 上传文件  
            minioClient.putObject(PutObjectArgs.builder()  
                    .bucket(bucketName)  
                    .object(fileName)  
                    .stream(file.getInputStream(), file.getSize(), -1)  
                    .contentType(file.getContentType())  
                    .build());  
              
            // 文件的完整路径名称  
            fileUrl = endpointUrl + "/" + bucketName + "/" + fileName;  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        return Result.ok(fileUrl);  
    }  
}
```

## spu 保存
1. 回显销售属性 baseSaleAttr
2. 保存spuInfo
```Java
/**  
 * 获取所有销售属性  
 *  
 * @return  
 */  
@Override  
public List<BaseSaleAttr> getSaleAttrList() {  
    return baseSaleAttrMapper.selectList(null);  
}  
  
  
/**  
 * 保存SPU(商品最小聚合信息)  
 * * @param spuInfo 包含(销售属性/商品图片/商品海报等)一种商品的信息  
 */  
@Transactional(rollbackFor = Exception.class)  
@Override  
public void saveSpuInfo(SpuInfo spuInfo) {  
      
    spuInfoMapper.insert(spuInfo);  
    Long spuInfoId = spuInfo.getId();  
      
    // 保存商品图片  
    List<SpuImage> spuImageList = spuInfo.getSpuImageList();  
    if (spuImageList != null) {  
        spuImageList.forEach(item -> {  
            item.setSpuId(spuInfoId);  
            spuImageMapper.insert(item);  
        });  
    }  
      
    // 保存商品海报  
    List<SpuPoster> spuPosterList = spuInfo.getSpuPosterList();  
    if (spuPosterList != null) {  
        spuPosterList.forEach(item -> {  
            item.setSpuId(spuInfoId);  
            spuPosterMapper.insert(item);  
        });  
    }  
      
    // 保存销售属性  
    List<SpuSaleAttr> spuSaleAttrList = spuInfo.getSpuSaleAttrList();  
    if (spuSaleAttrList != null) {  
        spuSaleAttrList.forEach(spuSaleAttr -> {  
            spuSaleAttr.setSpuId(spuInfoId);  
            spuSaleAttrMapper.insert(spuSaleAttr);  
              
            // 从每个销售属性中获取对应的销售属性值,添加到销售属性值表(spu_sale_attr_value)中  
            List<SpuSaleAttrValue> spuSaleAttrValueList = spuSaleAttr.getSpuSaleAttrValueList();  
            if (spuSaleAttrValueList != null) {  
                spuSaleAttrValueList.forEach(spuSaleAttrValue -> {  
                    spuSaleAttrValue.setSpuId(spuInfoId);  
                    spuSaleAttrValue.setSaleAttrValueName(spuSaleAttr.getSaleAttrName());  
                    spuSaleAttrValueMapper.insert(spuSaleAttrValue);  
                });  
            }  
        });  
    }  
}
```


# SKU
##  制作 sku 分析
spu + 销售属性 = sku
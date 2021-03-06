Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
	    <groupId>com.baoquan</groupId>
	    <artifactId>eagle-sdk</artifactId>
	    <version>2.0.16</version>
	</dependency>

如果使用gradle，可以加入如下依赖::
	
	compile group: 'com.baoquan', name: 'eagle-sdk', version: '2.0.16'

初始化客户端
------------------

::

	BaoquanClient client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	client.setHost("https://www.baoquan.com");
	// 设置access key
	client.setAccessKey("fsBswNzfECKZH9aWyh47fc"); 
	// 设置rsa私钥文件的绝对路径
	client.setPemPath("path/to/rsa_private.pem");
	// 设置版本
	client.setVersion("v2");

rsa私钥文件应该以 **-----BEGIN PRIVATE KEY-----** 开头和 **-----END PRIVATE KEY-----** 结尾，比如::

	-----BEGIN PRIVATE KEY-----
	MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBALS8adG98pHSLEq6
	kOT6PG25GMBzpiSs1oXwnPLTOVOYarffF0xSB7nk5yxbqx5BseJNz2NxyTpeJOk8
	FXEI7qTbS6oYAgyH/2HMr5Az3pKGLRdIjJQrpu3qpJkzRw82qGP2MkmVkUYeOl9B
	ZEUpk1GmziwrhbD0zcJITA0mnUqnAgMBAAECgYBnetUPjLTcvrwzURxyrb95hxff
	4JdIuljdOUVzVnKlJUg83JOHVBQuYBvn7thLq4uAqdJK+rQfIhX6IDeaj2WqsO7Y
	d4YoVxFAlfaHIICJKur15KOXuPMpdm3ilZ0c2yCTrJ0m3Xm6mpwd4blDDSupmlj4
	HEXXiInGZgwfTqONAQJBAOlX3EyvE2NvzYMh39wz11fmOi0UiyIvz0immjed4dhV
	0YvPjx8Gj7XGwCkzbuNwr7tlkMTaSiYR8cw1QzV4QoECQQDGSOlgAJC8oUP2+u4H
	+A83jfSLlhQ8XKAJn5Din9kBvs4eKMSjTpJiDBgA7NUAhUfCqS2/m5TiTiS3X3Ij
	ZKknAkEA2iaQCQks4SvnQI9s0FuPGdhdz0ODiCSWb9+CEjkCqdQhococje7+b/0u
	Ldat9uilAlfD7qX96HWiTz4EZXrXAQJBAJ+CbgMl0Ul9bcBUsoHEovEtCEn2TIcW
	eEPlkldNAfSuev+2CiHZhlbLpc+wtdU6YrUNBdl7HjVDabP+W0JvqscCQQDBoUR8
	Y3NUOdGRcaSgwT56tP5J1cZxg1b4vCyr+YfvcEGSBrEaxEugDUjxbON4etMVflh/
	H3QNSvRf4XQ44wQO
	-----END PRIVATE KEY-----

其它初始化设置
^^^^^^^^^^^^^^^

还有些其它可选的初始化设置，比如设置api版本，设置request id生成器，默认情况下你无需进行这些设置::

	// 设置request id生成器，生成器需要实现RequestIdGenerator接口中的createRequestId方法
	client.setRequestIdGenerator(CustomRequestGenerator) 

	// sdk中默认的request id生成器
	public class DefaultRequestIdGenerator implements RequestIdGenerator {

		@Override
		public String createRequestId() {
			return UUID.randomUUID().toString();
		}
	}

客户端初始化完成后即可调用客户端中的方法发送请求

创建保全
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	// 设置保全唯一码
	payload.setUniqueId("e68eb8bc-3d7a-4e22-be47-d7999fb40c9a");
	// 设置模板id
	payload.setTemplateId("5Yhus2mVSMnQRXobRJCYgt");
	// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
	payload.setCompleted(false);
	// 设置保全所有者的身份标识，标识类型定义在IdentityType中
	Map<IdentityType, String> identities = new HashMap<>();
	identities.put(IdentityType.ID, "42012319800127691X");
	identities.put(IdentityType.MO, "15857112383");
	payload.setIdentities(identities);
	// 添加陈述对象列表
	List<Factoid> factoids = new ArrayList<>();
	// 添加product陈述
	Factoid factoid = new Factoid();
	Product product = new Product();
	product.setName("xxx科技有限公司");
	product.setDescription("p2g理财平台");
	factoid.setUnique_id("e13912e2-ccce-47df-997a-9f44eb2c7b6c");
	factoid.setType("product");
	factoid.setData(product);
	factoids.add(factoid);
	// 添加user陈述
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setUnique_id("5bf54bc4-ec69-4a5d-b6e4-a3f670f795f3");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		CreateAttestationResponse response = client.createAttestation(payload);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

如果创建保全时需要给陈述上传对应的附件::

	// 创建3个附件，每个附件都是ByteArrayBody实例，ContentType必须为DEFAULT_BINARY，并且需要设置filename
	InputStream inputStream0 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody0 = new ByteArrayBody(IOUtils.toByteArray(inputStream0), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream1 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody1 = new ByteArrayBody(IOUtils.toByteArray(inputStream1), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream2 = getClass().getClassLoader().getResourceAsStream("contract.pdf");
	ByteArrayBody byteArrayBody2 = new ByteArrayBody(IOUtils.toByteArray(inputStream2), ContentType.DEFAULT_BINARY, "contract.pdf");
	// 创建附件map，key为factoids中的角标，此处设置factoids中第1个factoid有1个附件，第2个factoid有2两个附件
	Map<String, List<ByteArrayBody>> attachments = new HashMap<>();
	attachments.put("0", Collections.singletonList(byteArrayBody0));
	attachments.put("1", Arrays.asList(byteArrayBody1, byteArrayBody2));
	// 此处省略payload的创建
	try {
		CreateAttestationResponse response = client.createAttestation(payload, attachments);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述
------------------

::

	AddFactoidsPayload addFactoidsPayload = new AddFactoidsPayload();
	// 设置保全号
	addFactoidsPayload.setAno("7F189BBB5FA1451EA8601D0693E36FE7");
	// 添加陈述对象
	factoids = new ArrayList<>();
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setUnique_id("5bf54bc4-ec69-4a5d-b6e4-a3f670f795f3");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	addFactoidsPayload.setFactoids(factoids);
	// 调用追加陈述接口，如果成功则返回的success为true，如果失败则返回失败消息
	try {
		AddFactoidsResponse response = client.addFactoids(addFactoidsPayload);
		System.out.println(response.getData().isSuccess());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述的时候同样能为陈述上传附件，跟创建保全为陈述上传附件一样。

创建保全(sha256)
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	//模板必须为系统提供的文件HASH模板的子模板。
	payload.setTemplateId("filehash");
	payload.setUniqueId(randomUniqueId());
	Map<IdentityType, String> identities = new HashMap<IdentityType, String>();
	identities.put(IdentityType.MO, "15857110000");
	payload.setIdentities(identities);
	List<Factoid> factoids = new ArrayList<Factoid>();
	Factoid factoid = new Factoid();
	factoid.setUnique_id(randomUniqueId());
	factoid.setType("file");
	Map<String,String> map = new HashMap<String, String>();
	factoid.setData(map);
	map.put("owner_name","李三");
	map.put("owner_id","330124199501017791");
	factoids.add(factoid);
	payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		String sha256 = "654c71176b207401445fdd471f5e023f65af50d7361bf828e5b1c19c89b977b0";
		CreateAttestationResponse response = client.createAttestationWithSha256(payload,sha256);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

网页取证
------------------

::

    public void createAttestationWithUrl() throws ServerException, InterruptedException {
        String url = "http://www.qq.com/";
        CreateAttestation4UrlPayload payload = new CreateAttestation4UrlPayload();
        // 设置保全唯一码
        payload.setUniqueId(UUID.randomUUID().toString());
        // 设置模板id
        payload.setTemplateId("4g8kLrgrr8AGTXKqUzW1rc");
        // 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
        payload.setCompleted(true);
        // 设置保全所有者的身份标识，标识类型定义在IdentityType中

        Map<IdentityType, String> identities = new HashMap<IdentityType, String>();
        identities.put(IdentityType.ID, "429006198507104214");
        identities.put(IdentityType.MO, "18767106890");
        payload.setIdentities(identities);
        // 设置模板陈述信息
        List<Factoid> factoids = new ArrayList<Factoid>();
        Factoid qqxxFactoid = new Factoid();
        qqxxFactoid.setUnique_id(UUID.randomUUID().toString() + new Date().getTime());

        qqxxFactoid.setType("qqxx");
        payload.setUrl(url);
        Map<String, String> loanDataMap = new HashMap<String, String>();
        qqxxFactoid.setData(loanDataMap);
        loanDataMap.put("url", url);
        qqxxFactoid.setUnique_id(randomUniqueId());
        qqxxFactoid.setType("website");
        qqxxFactoid.setData(loanDataMap);
        factoids.add(qqxxFactoid);
        payload.setFactoids(factoids);

        // 设置网页取证相关信息
        payload.setUrl("http://www.simplechain.com/");
        payload.setLabel("简单上链");
        payload.setRemark("简单上链");
        payload.setWebName("简单上链");

        CreateAttestationResponse response = client.createAsyAttestationWithUrl(payload);
        System.out.print(response.getData().getNo());
    }

网页取证状态查询
------------------

::

    public void getAttestationUrl() throws ServerException {
        GetAttestationUrlResponse response = client.getAttestationWithUrl("E4276C1F61084FCA9AE1CB714526ACC9");
        System.out.print(response.getData().getStatus());
    }

获取保全数据
------------------

::

	try {
		GetAttestationResponse response = client.getAttestation("DB0C8DB14E3C44C7B9FBBE30EB179241", null);
		System.out.println(response.getData());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

getAttestation有两个参数，第1个参数ano是保全号，第二个参数fields是一个数组用于设置可选的返回字段

下载保全文件
------------------

::

	try {
		DownloadFile downloadFile = client.downloadAttestation("7FF4E8F6A6764CD0895146581B2B28AA");

		FileOutputStream fileOutputStream = new FileOutputStream(downloadFile.getFileName());
		IOUtils.copy(downloadFile.getFile(), fileOutputStream);
		fileOutputStream.close();
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}


用户认证信息同步
------------------

::

    try {
        UserKycResponse response = client.userKyc("15822222224", "用户一", "42012319800127691X");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

企业认证信息同步
------------------

::

	try {
		 KycEnterprisePayload payload = new KycEnterprisePayload();
        payload.setAccountName("潇潇公司");
        payload.setBank("中国银行");
        payload.setBankAccount("111111111111");
        payload.setName("这是我的公司");
        payload.setOrgcode("123456");
        payload.setPhone("17696526111");
        InputStream businessInputStream = getClass().getClassLoader().getResourceAsStream("licence.jpg");
        ByteArrayBody businessFile = new ByteArrayBody(IOUtils.toByteArray(businessInputStream), ContentType.DEFAULT_BINARY, "licence.jpg");
        kycEnterpriseResponse response = client.kycEnterprise(payload, businessFile);
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}



上传签章图片
------------------

::

    try {
         ContractPayload payload = new ContractPayload();
         InputStream inputStream = getClass().getClassLoader().getResourceAsStream("seal.png");
         ByteArrayBody byteArrayBody = new ByteArrayBody(IOUtils.toByteArray(inputStream), ContentType.DEFAULT_BINARY, "seal.png");
         Map<String, List<ByteArrayBody>> byteStreamBodyMap = new HashMap<String, List<ByteArrayBody>>();
         byteStreamBodyMap.put("0", Collections.singletonList(byteArrayBody));
         UploadSignatureResponse u=client.uploadSignature(payload, byteStreamBodyMap);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

设置默认签章图片
------------------

::

    try {
         SignaturePayload payload = new SignaturePayload();
         payload.setSignature_id("cey4FBLpqbsUNaLp3SENdp");
         client.setSignatureDefaultId(payload);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

签章图片列表
------------------

::

    try {
          client.listSignature();
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

上传合同
------------------

::

    try {
          ContractPayload payload = new ContractPayload();
          Calendar calendar = Calendar.getInstance();
          Date date = new Date(System.currentTimeMillis());
          calendar.setTime(date);
          calendar.add(Calendar.YEAR, +1);
          date = calendar.getTime();
          System.out.println(date);
          payload.setEnd_at(date);
          payload.setRemark("sas");
          payload.setTitle("ssss合同");
          InputStream inputStream = getClass().getClassLoader().getResourceAsStream("contract.pdf");
          ByteArrayBody byteArrayBody = new ByteArrayBody(IOUtils.toByteArray(inputStream), ContentType.DEFAULT_BINARY, "contract.pdf");
          Map<String, List<ByteArrayBody>> byteStreamBodyMap = new HashMap<String, List<ByteArrayBody>>();
          byteStreamBodyMap.put("0", Collections.singletonList(byteArrayBody));
          UploadContractResponse u = client.uploadContract(payload, byteStreamBodyMap);
          System.out.println(u.getContractId());
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }


发送验证码
------------------

::

    try {
          client.sendVerifyCode("hspH56P7nZU4XSJRWWGvpy", "15811111111","personal");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

签署合同或设置合同状态
------------------

::

    try {
        Map<String, String> identitiesMap = new HashMap<String, String>();
        List<PayloadFactoid> list = new ArrayList<PayloadFactoid>();
        PayloadFactoid payloadFactoid = new PayloadFactoid();
        LinkedHashMap<String , Object> linkedHashMap = new LinkedHashMap<String, Object>();
        linkedHashMap.put("userTruename","张三");
        linkedHashMap.put("address", "hangzhou");
        payloadFactoid.setType("product");
        payloadFactoid.setData(linkedHashMap);
        list.add(payloadFactoid);
        identitiesMap.put("MO", "15611111111");
        identitiesMap.put("ID", "430426198401361452");
        client.signContract("2B5KcmMKg195rHhLBuNbZB", "15611111111", "5755", "DONE", "4", "400", "550","_priv_template_2", identitiesMap, list,false,"","enterprise");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
	
合同列表
------------------

::

    try {
       ContractListPayload payload = new ContractListPayload();
        payload.setStatus("DONE");
        client.queryList(payload);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
	
合同签署详情信息
------------------

::

    try {
       client.getDetail("uqg9hB2JQg61g22ma2bFY2");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
    
签署合同下载
------------------
::

	try {
		DownloadFile downloadFile = client.downloadContract("jVef7CWtiFTvGRZ9ZG6ndD");

		FileOutputStream fileOutputStream = new FileOutputStream(downloadFile.getFileName());
		IOUtils.copy(downloadFile.getFile(), fileOutputStream);
		fileOutputStream.close();
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}


证据固定保全存证
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	// 设置保全唯一码
	payload.setUniqueId("e68eb8bc-3d7a-4e22-be47-d7999fb40c9a");
	// 设置模板id
	payload.setTemplateId("5Yhus2mVSMnQRXobRJCYgt");
	// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
	payload.setCompleted(true);
	// 设置保全所有者的身份标识，标识类型定义在IdentityType中
	Map<IdentityType, String> identities = new HashMap<>();
	identities.put(IdentityType.ID, "42012319800127691X");
	identities.put(IdentityType.MO, "15857112383");
	payload.setIdentities(identities);
	// 添加证据固定陈述对象列表
	List<Factoid> factoids = new ArrayList<>();
	// 添加qqxx陈述
	Factoid factoid = new Factoid();
	factoid.setUnique_id("e13912e2-ccce-47df-997a-9f44eb2c7b6c");
	factoid.setType("qqxx"); //这里type必须为"qqxx"
	Map<String, String> qqxxDataMap = new HashMap<String, String>();
        qqxxFactoid.setData(loanDataMap);
        qqxxDataMap.put("platFormId", "1");
	qqxxDataMap.put("nickname", "用户昵称");
        qqxxDataMap.put("ywlj", "https://www.baoquan.com/");
        qqxxDataMap.put("ywbt", "XX原文标题");
        qqxxDataMap.put("originalType","1");
        qqxxDataMap.put("url", "http://xx.com");
        qqxxDataMap.put("qqbt", "XX侵权标题");
        qqxxDataMap.put("qqwz", "XX网");
        qqxxDataMap.put("bqgs", "数秦科技");
	qqxxDataMap.put("cardNo", "342425199111103230");//证件号码（身份证或者统一社会代码）
        qqxxDataMap.put("qqbh", "qq001");
        qqxxDataMap.put("qqzt", "XX网");
	qqxxDataMap.put("oriSubDate", "2018-01-01 06:20");//原文发布时间
	qqxxDataMap.put("pirSubDate", "2018-01-02 06:20");//侵权文章发布时间
        qqxxDataMap.put("qqzt", "XX网");
        qqxxDataMap.put("matchNum", "0.99");
        factoids.add(qqxxFactoid);
        payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		CreateAttestationResponse response = client.fixedEvidence(payload);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

添加原创
------------------

::

	OriginalArticlePayload payload = new OriginalArticlePayload();
        // 设置原创认证唯一码
        payload.setUniqueId(UUID.randomUUID().toString());
        payload.setLinkUrl("http://www.baidu.com");
        payload.setNickName("1111");
        payload.setOriginalType("1,2");
        payload.setPlatformCode("1");
        payload.setSubDate("2018-06-27 15:22");
        payload.setTitle("文章标题");
	// 调用添加原创接口，如果成功则返回原创文章Id，如果失败则返回失败消息
	try {
		OriginalArticleResponse response = client.createOriginalArticle(payload);
		 System.out.print(response.getOriginalId());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

客户免验证码签约授权发送验证码
------------------

::

	   public void testSendAuthorizationVerifyCode() throws ServerException {
        client.sendAuthorizationVerifyCode("15811111111", "personal");
    }

客户免验证码签约授权确认
------------------

::

    public void testauthorized() throws ServerException {
        client.authorized("15811111111", "7333", "personal");
    }


---
title: Vue axios + Spring boot 이미지 업로드 하기
tags: 
 - Spring
 - Vue
 key: 1
---

### Vue axios + Spring boot 이미지 업로드 하기

웹 페이지에서 이미지 또는 파일 업로드가 필요한 기능들이 많이 있습니다.  이미지 업로드에는 다양한 방법이 있으나  vue에서 axios를 사용하여 spring boot의 rest api를 사용하여 서버에 이미지를 업로드를 해보겠습니다.



##### Vue 코드 

ImageUpload.vue

```vue
<template>
	<div>
		<input type="file" accept="image/*" multiple @change="handleChange" />
		<button @click="uploadFiles">파일업로드</button>
	</div>
</template>

<script>
import axios from 'axios'
export default {
	data() {
		return {
			images: [],
		}
	},
	methods: {
		handleChange(event) {
			this.images = event.srcElement.files
		},
		uploadFiles() {
			if (this.images.length == 0) {
				console.log('업로드 할 이미지가 없습니다.')
				return
			}
			const formData = new FormData()

			for (let i = 0; i < this.images.length; i++) {
				formData.append('images', this.images[i])
			}

			axios({
				url: 'http://localhost:8081/api/uploadImage',
				method: 'POST',
				data: formData,
			})
				.then(result => {
					console.log(result)
				})
				.catch(err => {
					console.log(err)
				})
		},
	},
}
</script>

<style></style>

```

multipart/form-data 타입으로 이미지를 전송하기 위해 FormData()를 사용합니다. 여러개의 이미지를 업로드 하더라도 동일한 images라는 이름으로 append를 합니다. 만약 이미지와 다른 데이터를 함께 전송하고 싶다면 formData에 함께 append를 해주면 됩니다. 



##### Spring Boot 

###### StaticResourceConfig.class

``` java
@Configuration
public class StaticResourceConfig extends WebMvcConfigurerAdapter {

    @Value("${static.resource.location}")
    private String staticResouceLocation;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/assets/**").addResourceLocations(staticResouceLocation);
    }
}
```

###### application.properties

``` properties
# 스프링의 주소를 사용하여 이미지를 불러오는 방법  http://localhost:8080/assets/'이미지 이름'
# 필요에 맞게 수정
static.resource.location=file:///C:/assets/
#static.resource.location=file:/home/assets/   리눅스 경로
```

Spring Boot가 배포되고 있는 서버에서 이미지를 저장하고 불러오기 위한 환경설정을 하는 설정입니다.

###### ImageUploadController.class

``` java
@RestController
@CrossOrigin("*")
@RequestMapping("/api")
public class ImageUploadController {
	@RequestMapping(path = "/uploadImage", method = RequestMethod.POST)
	public String uploadImage(HttpServletRequest request,@RequestPart MultipartFile[] images ) throws IOException {
		String title = request.getParameter("title");
		System.out.println(title);
		for (int i = 0; i < images.length; i++) {
			System.out.println("======================");
			System.out.println(images[i].getName());
			System.out.println(images[i].getOriginalFilename());
			System.out.println(images[i].getContentType());
			System.out.println("======================");
			
			String filename = images[i].getOriginalFilename();
			byte[] imageData = images[i].getBytes();
			File newFile = null;
			FileOutputStream fileOutputStream = null;
			
			String url ="";
			try {
                // 이미지 저장 폴더가 존재하지 않다면 생성
				File dir = new File("C:\\assets/");
                // File dir = new File("/home/assets/");
				if(!dir.isDirectory()) {
					dir.mkdir();
				}
				
				// 업로드 된 이미지 저장
				newFile = new File("C:\\assets/"+filename);
                // newFile = new File("/home/assets/"+filename);
				fileOutputStream = new FileOutputStream(newFile);
				fileOutputStream.write(imageData);
			} catch (Throwable e) {
				e.printStackTrace(System.out);
				return "failure";
			} finally {
				fileOutputStream.close();
				url = "http://localhost:8081/assets/"+filename;
				System.out.println(url);
			}
		}
		return "success";
	}
}
```

vue에서 axios로 전송한 이미지를 받기 위해 `@RequestPart MultipartFile[] images` array 형태로 데이터를 받습니다. images 의 이름은 동일해야 합니다. 이미지 외의 다른 데이터는 `HttpServletRequest request`의 `request.getParameter();` 를 사용하여 받을 수 있습니다.

저장된 이미지를 요청하고 싶다면  url을 사용하여 요청할 수 있습니다.


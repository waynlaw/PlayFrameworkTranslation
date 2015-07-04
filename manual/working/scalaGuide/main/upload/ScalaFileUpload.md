<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 파일 업로드 다루기

## 폼에서 multipart/form-data를 사용하여 파일 업로드하기

웹 애플리케이션에서 파일들을 업로드 하는 일반적인 방법은, 첨부하는 파일의 데이터와 일반적인 폼의 데이터를 섞을 수 있는 특별한 `multipart/form-data` 인코딩을 사용하는 폼을 이용하는 것이다. 주의해야 할 점은 폼을 제출하는 HTTP 방식이 POST를 사용해야 한다는 점이다. (GET이 아니다.)

HTML 폼을 작성해 보자.

@[file-upload-form](code/scalaguide/templates/views/uploadForm.scala.html)


이제 `upload` 액션을 `multipartFormData` 내용 파서를 사용해서 정의하자.

@[upload-file-action](code/ScalaFileUpload.scala)


`ref` 속성은 `TemporaryFile`의 참조를 제공해준다. 이것은 `mutipartFormData` 파서가 파일 업로드를 다루는 기본적인 방식이다.

> **주의:** 다른 경우와 마찬가지로, `anyContent` 내용 파서를 사용하거나, 그걸 `request.body.asMultipartFormData`처럼 가져올 수 있다.

마지막으로 POST 라우터를 추가하자.

@[application-upload-routes](code/scalaguide.upload.fileupload.routes)


## 직접 파일 업로드 하기

파일들을 서버로 전송하는 다른 방법은 폼에서 파일들을 비동기로 업로드 하기위해 Ajax를 사용하는 것이다. 이 경우에는 요청 몸체가 `multipart/form-data`로 인코딩 되지않고, 파일 내용 그대로가 될 것이다.

이 경우에는 단지 파일에서 내용을 보관하는 용도로 내용 파서를 사용할 수 있다. `TemporaryFile` 내용 파서를 사용하는 예는 아래와 같다.

@[upload-file-directly-action](code/ScalaFileUpload.scala)

## 고유의 내용 파서 작성하기

만일 파일 업로드를 임시파일에 보관하지 않고 직접 다루기를 원하면, 고유한 `BodyParser`를 작성해야 한다. 이 경우에는, 원하는 곳으로 전달할 수 있는 데이터 묶음을 전달 받을 것이다.

만일 `multipart/form-data` 인코딩 형식을 사용하기를 원하면, 고유한 `PartHandler[FilePart[A]]`를 작성하여 기본 `mutipartFormData` 파서에 전달하여 사용할 수 있다. 그러면 부분 헤더를 전달 받아서, 올바른 `FilerPart`를 만드는 `Iteratee[Array[Byte], FilePart[A]]`를 제공해야 할 것이다.

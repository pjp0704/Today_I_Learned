# Tabnabbing 피싱 공격

탭내빙(Tabnabbing) 공격은 피싱 공격 기법 중 하나로, 사용자가 눈치채지 못하는 사이에 사용하작 원래 보고 있던 부모 탭을 변조하여 원래 사이트와 유사한 피싱 페이지로 리다이렉트할 수 있는 공격이다.

## 탭내빙(Tabnabbing) 공격 원리

HTML 문서 내에서 링크(target이 \_blank인 태그)를 클릭했을 때 새롭게 열린 탭 또는 페이지에서 기존 문서인 location을 피싱 사이트로 변경해 정보를 탈취하는 것이 기본적인 원리이다.

![tabnabbing_example](https://blog.jxck.io/entries/2016-06-12/tab-nabbing.svg?180105_115707#500x500)

> 출처: blog.jxck.io

사진의 예시를 순서대로 설명하자면:

1. 사용자가 cgm.example.com에 접속한다.
2. 해당 사이트에서 happy.example.com으로 갈 수 있는 **target이 \_blank인 링크 태그**를 클릭한다.
3. 새 탭에서 happy.example.com이 열린다.
   - 이 때 happy.example.com에는 window.opener 속성이 존재하며,
   - 자바스크립트를 이용해 opener의 location을 피싱 사이트인 cg**n**.example.com으로 변경한다.
4. 사용자가 다시 원래의 탭으로 돌아온다.
5. 로그인이 풀린 것처럼 착각하게 하여 아이디와 비밀번호를 입력하도록 유도한다.
   - cgn.example.com은 사용자가 입력한 계정 정보를 탈취한 후 다시 본래의 사이트로 리다이렉트시킨다.

![tabnabbing_example_with_navermail](https://tech.lezhin.com/files/2017-06-tabnabbing/naver.gif)

> 출처: 레진 기술 블로그(tech.lezhin.com)

## 방어할 수 있는 방법은?

### rel="noopener" 속성 사용

noopener 속성을 이용하면 참조자 정보를 보내지 않아 window.opener가 무효화된다. 따라서 noopener 속성을 가진 링크를 통해 열린 페이지는 window.opener를 통해 부모 탭을 참조할 수 없고, location과 같은 자바스크립트 요청을 거부하게 된다.

```html
<a href="http://example.com" target="_blank" rel="noopener"> Example Link </a>
```

### rel="noreferrer" 속성 사용

noreferrer 속성을 이용하면 링크를 통해 접근 시 포함된 referrer를 전송하지 않도록 하기 때문에 링크를 클릭하더라도 referrer 정보가 유출되지 않는다.

```html
<a href="http://example.com" target="_blank" rel="noreferrer"> Example Link </a>
```

### rel="nofollow" 속성 사용

nofollow는 사용 중인 브라우저에서 문서에 포함된 링크로 따라가지 않도록 지정하는 속성으로 모든 링크를 따라가지 않게 하려면 meta 요소에 nofollow를 지정하고 개별적으로 지정하려면 href 해당 요소에 지정한다.

```html
<a href="http://example.com" target="_blank" rel="nofollow"> Example Link </a>
```

#### 참고 자료

레진 기술 블로그: <https://tech.lezhin.com/2017/06/12/tabnabbing>
이글루시큐리티: <http://www.igloosec.co.kr/BLOG_Tabnabbing%20기법을%20통한%20계정%20탈취?searchItem=&searchWord=&bbsCateId=0&gotoPage=18>

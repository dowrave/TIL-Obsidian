- 의외로 백엔드를 써야 하는 영역이었음

## 1. 백엔드

### Models.py
- `Subsection` 모델에 두 필드를 추가했다.
```python
    background_color = models.CharField(max_length = 7, default = '#FFFFFF')
    text_color = models.CharField(max_length = 7, default='#000000')
```
> `hex Code`로 색을 지정해놨다. migrate는 상식이라 생략함.

### views.py, serializers.py, urls.py
```python
class SubsectionEditColorView(APIView):

    permission_classes = [IsAuthenticated]
    authentication_classes = [JWTAuthentication]

    def post(self, request, *args, **kwargs):
        
        subsection_req = request.data.get('subsection')
        subsection_obj = Subsection.objects.get(name = subsection_req)
        serializer = PostUpdateSerializer(subsection_obj, # 기존 객체
                            data = request, # 업데이트할 내용
                            context = {'request' : request} # serializer에서 request 객체에 접근 가능하게 함
                            )
        if serializer.is_valid():
            serializer.save() # 이는 결국 models.py의 save 메서드를 호출함
            return Response(serializer.data, status = status.HTTP_200_OK)
        
        else:
            print(serializer.errors)
            return Response(serializer.errors,
                            status = status.HTTP_400_BAD_REQUEST)

# --- serializers.py
class SubsectionEditColorSerializer(serializers.Modelserializer):
    class Meta:
        model = Subsection
        fields = '__all__'

# --- urls.py
    re_path(r'^subsection/edit/color$', SubsectionEditColorView.as_view(), name = 'SubsectionEditColor'),
```
> 기존 `PostEditView`를 참고해서 작성함

## 2. 프론트엔드

### App.tsx - 모든 subsection에 스타일 적용하기
- 모든 subsection의 스타일을 가져옴
```tsx
  useEffect(() => {
    /**
     * 백엔드에서 모든 subsection의 배경색, 글자색을 가져와 지정함
     */
    const initSubsectionStyle = async () => {
      try {
        const response = await axios.get(backend + 'api/blog/subsection/')
        const datas = response.data.subsection 

        datas.forEach(data => {
          const subsectionId = data.subsection
          const backgroundColor = data.background_color;
          const textColor = data.text_color;

          dispatch(setSubsectionStyle({ subsectionId, 
                                        style : { backgroundColor, textColor }}))
  
        })

      } catch (e) {
        console.log('에러 발생 : ', e)
      }
    }

    initSubsectionStyle();
  }, [])  

  useEffect(() => {
    console.log('reduxStyles : ', reduxStyles)
  }, [reduxStyles])
```

- Redux로 상태를 저장하고 모든 값을 저장했음
### stylesReducer.ts
```tsx
const SET_SUBSECTION_STYLE = 'SET_SUBSECTION_STYLE';

export const setSubsectionStyle = ({subsectionId, backgroundColor, textColor}) => ({
    type: SET_SUBSECTION_STYLE,
    payload: {subsectionId, 
              style: {backgroundColor, textColor}
                },
})

const initialState = {};

const stylesReducer = (state = initialState, action) => {
    switch (action.type) {
        case SET_SUBSECTION_STYLE:
            return {
                ...state,
                [action.payload.subsectionId] : {...action.payload.style}
            }
        default:
            return state;
    }
}

export default stylesReducer;
```



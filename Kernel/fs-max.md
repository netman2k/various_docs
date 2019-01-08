## file-max 및 사용되고 있는 File Descriptor 구하기
* file-max: File handler(혹은 File descriptor) 최대 수로 커널이 할당
* 계산 공식
```
PAGE_SIZE=$(getconf -a | grep PAGE_SIZE | awk '{print $2}')
MEM_PAGES=$(getconf -a |grep _PHYS_PAGES | awk '{print $2}')
FILE_MAX=$(($MEM_PAGES * $(($PAGE_SIZE / 1024)) / 10))
"Calculated file max: ${FILE_MAX}"
```
> 실제 시스템에 설정된 값은 위 계산식으로 구해진 값보다 작거나 같을 것이다. 

* 다음 커맨드는 현재 프로세스가 사용되고 있는 파일 디스크립터를 확인할 수 있다
```
for pid in $(lsof | awk '{ print $2 }' | uniq) 
do 
    find /proc/$pid/fd/pe l 2>&1 | grep -v "No" 
done
```

### Reference
* [what are file-max and file-nr linux kernel parameters?](http://www.linuxvox.com/post/what-are-file-max-and-file-nr-linux-kernel-parameters/)
* [리눅스 서버의 TCP 네트워크 성능을 결정짓는 커널 파라미터 이야기 - 2편](https://meetup.toast.com/posts/54)

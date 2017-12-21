## Squashing commits with rebase
[Origin](http://ko.gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html)

**--interactive** (혹은 **-i**) 모드용 막강한 옵션들을 보유한 rebase 명령은, 커밋들을 합치고 싶을 때 가장 널리 사용되는 명령 중 하나이다. 이 명령이 하는 일은 작은 커밋들을 결합하여 하나의 큰 커밋으로 만드는 것이며, 만약 그날 한 일을 마무리지으려 하거나 아니면 그냥 서로 다른 변경사항들을 묶고 싶어졌을 때 유용하다. 우리는 이제 이런 일들을 어떻게 쉽게 할 수 있는지 알아볼 것이다.

> *주의사항 한마디: 오직 외부 저장소에 push 되지 않은 커밋에 대해서만 이 기능을 사용하라. 만약 다른 사람들이 당신이 삭제하려고 하는 커밋에 기반을 둔 작업을 하고 있다면, 꽤 많은 충돌이 발생할 것이다. 쉽게 말해 다른 사람과 공유하고 있는 역사를 고쳐쓰지 말라.*

그럼 당신이 커밋을 몇번 했고, 그것들을 합쳐서 큰 커밋 하나로 만들고 싶다고 하자. 우리 저장소의 역사는 현재 다음과 같다.

![](images/squash1.png)

이 최근 4개의 커밋들은 하나로 합쳐지면 더 적절할 것 같으니, interactive rebasing으로 그걸 해보자:
```
$ git rebase -i HEAD~4

pick 01d1124 Adding license
pick 6340aaa Moving license into its own file
pick ebfd367 Jekyll has become self-aware.
pick 30e0ccb Changed the tagline in the binary, too.

# Rebase 60709da..30e0ccb onto 60709da
#
# Commands:
#  p, pick = use commit
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

자, 몇가지 일들이 여기서 일어났다. 우선, 나는 Git에게 HEAD 부터 HEAD~4 까지 지난 네 커밋을 rebase 하라고 시켰다. Git는 이제 무엇을 할 수 있는지에 대한 약간의 설명을 포함한 위의 텍스트와 함께 나를 편집기로 보냈다. 화면에 여러가지 옵션들이 나타났지만, 지금 우리는 그냥 커밋들을 하나로 합치는 것만 할 것이다. 자, 위 파일의 첫 네 줄을 이렇게 바꾸면 된다:

```
pick 01d1124 Adding license
squash 6340aaa Moving license into its own file
squash ebfd367 Jekyll has become self-aware.
squash 30e0ccb Changed the tagline in the binary, too.
```
기본적으로 이것은 Git에게 나열된 네 커밋을 하나의 커밋으로 합치도록 시킨다. 일단 저장하면 다음과 같이 또 에디터가 실행된다:
```
# This is a combination of 4 commits.
# The first commit's message is:
Adding license

# This is the 2nd commit message:

Moving license into its own file

# This is the 3rd commit message:

Jekyll has become self-aware.

# This is the 4th commit message:

Changed the tagline in the binary, too.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# Explicit paths specified without -i nor -o; assuming --only paths...
# Not currently on any branch.
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   LICENSE
#	modified:   README.textile
#	modified:   Rakefile
#	modified:   bin/jekyll
#
```
우리가 여러 커밋들을 합쳤기 때문에, Git은 이 작업과 관련된 커밋들의 메시지로 새 커밋 메시지를 작성할 수 있게 해준다. 메시지를 원하는 대로 수정하고, 저장하고, 종료하라. 그렇게 하면 그 커밋들은 성공적으로 합쳐진다!
```
Created commit 0fc4eea: Creating license file, and making jekyll self-aware.
 4 files changed, 27 insertions(+), 30 deletions(-)
  create mode 100644 LICENSE
	Successfully rebased and updated refs/heads/master.
```
그리고 우리가 히스토리를 다시 들여다보면…

![](images/squash2.png)
 
이건 상대적으로 그다지 괴롭지 않다. 만약 rebase 중에 충돌 상태로 들어갔다면 그건 resolve 하기 상당히 쉽고 Git는 그걸 할 수 있도록 안내해 줄 것이다. 기본적으로 질문받은 충돌을 고치고, 그 파일을 git add하고, git rebase --continue 로 계속 진행하면 된다. 물론 원한다면 git rebase --abort 로 이전 상태로 돌아갈 수 있다. 어떤 이유로 인해 rebase 과정에서 커밋을 잃어버렸다면, reflog 를 사용해 되돌릴 수 있다.

여기서 다루지 않은 git rebase -i 의 사용법이 상당히 많다. 혹시 공유하고 싶은 것이 있다면, 부디 그렇게 해달라! GitCasts 에도, 이 과정 전반 및 더 복잡한 예제도 다루고 있는 환상적인 영상 이 올라와 있다.

 
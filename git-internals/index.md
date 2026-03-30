# Git 내부 구조 — blob, tree, commit 객체의 실체

Git은 단순한 버전 관리 도구가 아니라, 콘텐츠 주소 지정(content-addressable) 파일시스템 위에 구축된 도구 모음이다. `.git` 디렉토리 안의 객체 데이터베이스를 직접 들여다보며 Git의 본질을 이해해 보자.

## 객체 모델

Git의 모든 데이터는 네 종류의 객체로 저장된다. blob(파일 내용), tree(디렉토리), commit(스냅샷), tag(주석 태그). 각 객체는 SHA-1 해시로 고유하게 식별된다.

```bash
# 현재 저장소의 객체 수 확인
git count-objects -v
# count: 42
# size: 168
# in-pack: 1234
# packs: 1

# 객체 타입 확인
git cat-file -t abc1234
# commit

# 객체 내용 확인
git cat-file -p abc1234
```

모든 객체는 `.git/objects/` 디렉토리에 저장된다. 해시의 처음 2글자가 디렉토리명, 나머지가 파일명이 된다. zlib으로 압축되어 있다.

### blob 객체

blob은 파일의 내용만 저장한다. 파일명이나 퍼미션은 포함하지 않는다.

```bash
# 문자열로 blob 직접 생성
echo "Hello, Git!" | git hash-object -w --stdin
# 8d0e412cd0debc41c1b1cbe34c7a0c3d7b45a24d

# blob 내용 확인
git cat-file -p 8d0e41
# Hello, Git!

# 파일로 blob 생성
git hash-object -w README.md
```

같은 내용을 가진 파일은 아무리 많아도 하나의 blob만 저장된다. 이것이 Git의 저장 효율성의 기반이다.

## tree 객체

tree는 디렉토리 구조를 표현한다. 각 엔트리는 퍼미션, 객체 타입, 해시, 파일명으로 구성된다.

```bash
# 현재 HEAD의 루트 tree 확인
git cat-file -p HEAD^{tree}
# 100644 blob a1b2c3d4...  .gitignore
# 100644 blob e5f6a7b8...  README.md
# 040000 tree 1a2b3c4d...  src
# 040000 tree 5e6f7a8b...  styles

# 하위 tree 확인
git cat-file -p 1a2b3c4d
# 100644 blob 9c0d1e2f...  render.tsx
# 040000 tree abcdef12...  components
```

tree는 재귀적으로 다른 tree를 포함할 수 있어, 전체 디렉토리 계층을 표현한다. `100644`는 일반 파일, `040000`은 디렉토리, `100755`는 실행 파일을 의미한다.

### 저수준에서 tree 생성

```bash
# 스테이징 영역 없이 tree 직접 생성
git mktree <<EOF
100644 blob 8d0e412cd0debc41c1b1cbe34c7a0c3d7b45a24d	hello.txt
100644 blob a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0	world.txt
EOF
```

`git add`와 `git commit`은 결국 이런 저수준 연산의 조합이다.

## commit 객체

commit은 tree(스냅샷)에 메타데이터를 더한 것이다. tree 해시, 부모 commit 해시, 작성자, 커미터, 메시지를 포함한다.

```bash
git cat-file -p HEAD
# tree 5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f
# parent a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0
# author devlog <devlog@example.com> 1706000000 +0900
# committer devlog <devlog@example.com> 1706000000 +0900
#
# Add initial blog post structure
```

부모가 없으면 최초 커밋, 두 개면 머지 커밋이다. 각 커밋이 이전 커밋을 가리키는 단방향 링크드 리스트가 Git 히스토리의 본질이다.

### 커밋 직접 생성

```bash
# 저수준 명령으로 커밋 생성
TREE=$(git write-tree)
PARENT=$(git rev-parse HEAD)
COMMIT=$(echo "Manual commit" | git commit-tree $TREE -p $PARENT)
git update-ref refs/heads/main $COMMIT
```

이것이 `git commit`의 본질이다. write-tree로 스테이징 영역을 tree로 변환하고, commit-tree로 메타데이터를 추가하고, update-ref로 브랜치 포인터를 이동시킨다.

## 브랜치와 레퍼런스

브랜치는 커밋 해시를 가리키는 텍스트 파일에 불과하다. `.git/refs/heads/main`에 40자의 해시가 적혀있을 뿐이다.

```bash
cat .git/refs/heads/main
# a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0

cat .git/HEAD
# ref: refs/heads/main
```

`HEAD`는 현재 체크아웃된 브랜치를 가리키는 심볼릭 레퍼런스다. 브랜치 생성은 파일 하나를 만드는 것이고, 커밋은 이 파일의 내용을 새 해시로 업데이트하는 것이다.

### packfile

객체가 많아지면 Git은 loose 객체들을 packfile로 압축한다. 델타 압축을 적용하여 유사한 객체 간 차이만 저장하므로, 저장 공간을 극적으로 절약한다.

```bash
# 수동 pack
git gc

# pack 내용 확인
git verify-pack -v .git/objects/pack/pack-*.idx | head -5
```

`git gc`가 자동으로 실행되어 loose 객체를 pack하고, 도달 불가능한 객체를 정리한다.

## 실용적 활용

Git 내부 구조를 이해하면 복잡한 상황에서 당황하지 않을 수 있다. `reflog`으로 삭제된 커밋을 복구하거나, `fsck`로 저장소 무결성을 검증하거나, `filter-branch`(또는 `filter-repo`)로 히스토리를 재작성하는 것이 더 이상 마법이 아니게 된다.

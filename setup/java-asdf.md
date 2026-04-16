# Java (Temurin 21) 통합 설치 가이드 (macOS / Linux, zsh)

이 문서는 `zsh`를 쓰는 macOS/Linux 개발 환경에서 `asdf`로 Java(Temurin 21)를 설치하고, 프로젝트 버전을 고정하는 방법을 다룸

## 1. asdf 설치 & 설정

아래 링크를 참고
1. 설치: https://asdf-vm.com/guide/getting-started.html#_1-install-asdf
2. 설정: https://asdf-vm.com/guide/getting-started.html#_2-configure-asdf

## 2. Java 플러그인 추가 (최초 1회)

```zsh
asdf plugin add java https://github.com/halcyon/asdf-java.git
```

참고: https://github.com/halcyon/asdf-java

## 3. JDK 설치 (Temurin 21)

```zsh
asdf install java latest:temurin-21
```

설치 확인:

```zsh
asdf list java
```

## 4. JAVA_HOME 설정 (zsh)

`~/.zshrc` 또는 `~/.zshrc.local`에 아래를 추가

```zsh
asdf_update_java_home() {
  local java_path

  # asdf 미설정 디렉터리에서 경고 출력 억제
  java_path="$(asdf which java 2>/dev/null)"

  if [[ -n "$java_path" ]]; then
    export JAVA_HOME="$(dirname "$(dirname "${java_path:A}")")"
    export JDK_HOME="$JAVA_HOME"
  else
    # 이전 디렉터리의 값이 남지 않도록 정리
    unset JAVA_HOME
    unset JDK_HOME
  fi
}

autoload -U add-zsh-hook
add-zsh-hook precmd asdf_update_java_home
add-zsh-hook chpwd asdf_update_java_home

# 현재 셸에 즉시 반영
asdf_update_java_home
```

적용:

```zsh
source ~/.zshrc
echo $JAVA_HOME
```

확인:

```zsh
java -version
```

참고:
- https://github.com/halcyon/asdf-java/blob/master/set-java-home.zsh
- https://github.com/halcyon/asdf-java?tab=readme-ov-file#java_home

## 5. macOS 전용 JAVA_HOME 연동 (선택)

macOS에서 시스템 `JAVA_HOME` 연동이 필요하면 `~/.asdfrc`에 설정

```zsh
java_macos_integration_enable=yes
```

참고: https://github.com/halcyon/asdf-java?tab=readme-ov-file#macos

## 6. 프로젝트 버전 고정

프로젝트 루트에서 실행:

```zsh
asdf set java latest:temurin-21
```

그러면 `.tool-versions`에 Java 버전이 기록

확인:

```zsh
cat .tool-versions
```

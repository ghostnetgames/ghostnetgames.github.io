---
title: Cobra & Viper 활용하기
permalink: mydoc_go_stingofviper.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Sting of the Viper (독사의 독침)
[sting of the viper](https://github.com/carolynvs/stingoftheviper/blob/main/README.md)글을 참고하였습니다. Cobra와 Viper를 혼합하여 사용하는 방법을 보여주고 있습니다. Cobra는 golang의 명령줄 인터페이스(CLI) Command-Line Interface 또는 Character User Interface를 쉽게 만들 수 있도록 해주는 패키지입니다. 반면에 Viper는 Config 파일을 바인딩할 수 있도록 해줍니다. 
이 예제 코드를 기반으로 사용법을 분석합니다.  
저장소의 코드를 가져옵니다. 

```shell
foo@bar:~$ go get github.com/carolynvs/stingoftheviper
# or (go 버전에 따라 안될 수 있다.)
foo@bar:~$ git clone https://github.com/carolynvs/stingoftheviper.git
foo@bar:~$ cd stingoftheviper/
```

```shell
$ go build .
$ go test ./... # test방법도 함께 분석해보겠습니다.
```

```shell
$ ./stingoftheviper
Your favorite color is: blue
The magic number is: 7
```

### Code를 분석해보자
전체 소스 코드는 [sting of the viper Github](https://github.com/carolynvs/stingoftheviper/blob/main/README.md) 를 참고하세요.
```go
const (
	defaultConfigFilename = "stingoftheviper"
	envPrefix = "STING"
)
```
`defaultConfigFilename`은 Config 파일 이름이다. 확장자가 없는 이유는 Viper가 대부분의 Config 파일들을 지원하기 때문이다. `envPrefix`는 모든 환경 변수의 접두사이다. 예를 들어 --number는 STING_NUMBER에 바인딩 된다.

```go
func NewRootCommand() *cobra.Command {
	color := ""
	number := 0

	// Define our command
	rootCmd := &cobra.Command{
		Use:   "stingoftheviper",
		Short: "Cober and Viper together at last",
		Long:  `Demonstrate how to get cobra flags to bind to viper properly`,
		PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
			// Cobra와 Viper를 바인딩하는 것은 PersistentPreRunE을 추천합니다.
			return initializeConfig(cmd)
		},
		Run: func(cmd *cobra.Command, args []string) {
			// Working with OutOrStdout/OutOrStderr allows us to unit test our command easier
			out := cmd.OutOrStdout()

			// Print the final resolved value from binding cobra flags and viper config
			fmt.Fprintln(out, "Your favorite color is:", color)
			fmt.Fprintln(out, "The magic number is:", number)
		},
	}

	// Define cobra flags, the default value has the lowest (least significant) precedence
	rootCmd.Flags().IntVarP(&number, "number", "n", 7, "What is the magic number?")
	rootCmd.Flags().StringVarP(&color, "favorite-color", "c", "red", "Should come from flag first, then env var STING_FAVORITE_COLOR then the config file, then the default last")

	return rootCmd
}
```
`cobra.Command`의 구조체를 채운다. 눈여겨 볼 것은 Function Handler를 등록하는 부분이다. PersistentPreRunE에 `initializeConfig`를 호출하도록 했는데 이 함수에서 Viper의 Config 바인딩을 시작한다. 그리고 Run은 실제 Command 값을 불러와서 최종 설정된 값을 호출한다. 그리고 `rootCmd.Flags()`를 통해 default값을 설정하는 데 우선 순위는 다음과 같다.  

1. Command Flag
1. 시스템 환경변수
1. Config파일
1. Default 값

```go

func initializeConfig(cmd *cobra.Command) error {
	v := viper.New()

	// Config파일을 등록합니다. 확장자는 입력하지 않습니다.
	v.SetConfigName(defaultConfigFilename)

	// Config 파일 Path입니다.
	v.AddConfigPath(".")

	// Attempt to read the config file, gracefully ignoring errors
	// caused by a config file not being found. Return an error
	// if we cannot parse the config file.
	if err := v.ReadInConfig(); err != nil {
		// It's okay if there isn't a config file
		if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
			return err
		}
	}

	// 환경 변수와 충돌을 피할 수 있도록 Prefix를 붙인다. Prefix를 제거된 상태로 변수와 바인딩된다.
	v.SetEnvPrefix(envPrefix)

	// 환경 변수와 바인딩한다. 
	v.AutomaticEnv()

	// command flag를 viper와 바인딩 한다.
	bindFlags(cmd, v)

	return nil
}
```

Viper는 시스템 환경변수와 Config 파일를 바인딩하는 기능을 제공한다. 
환경 변수는 '-' 문자를 갖을 수 없다. 이것을 '_'으로 변경하고 대문자로 변경후 Prefix를 붙여서 Viper에 바인딩한다. e.g. --favorite-color to STING_FAVORITE_COLOR

```go
// 각 cobra flag와 viper configuration등을 바인딩한다.
func bindFlags(cmd *cobra.Command, v *viper.Viper) {
	cmd.Flags().VisitAll(func(f *pflag.Flag) {
		// 환경 변수는 '-' 문자를 갖을 수 없다. 이것을 '_'으로 변경하고 대문자로 변경후 Prefix를 붙여서 
		// Viper에 바인딩한다. e.g. --favorite-color to STING_FAVORITE_COLOR
		if strings.Contains(f.Name, "-") {
			envVarSuffix := strings.ToUpper(strings.ReplaceAll(f.Name, "-", "_"))
			v.BindEnv(f.Name, fmt.Sprintf("%s_%s", envPrefix, envVarSuffix))
		}
		
		// flag에 없고 Viper에 있는 값을 Cobra에 바인딩한다. 
		if !f.Changed && v.IsSet(f.Name) {
			val := v.Get(f.Name)
			cmd.Flags().Set(f.Name, fmt.Sprintf("%v", val))
		}
	})
}
```
### Unit Test 파일을 분석해보자.
Unit Test를 작성하는 것은 생각보다 쉽지 않다. Function 하나가 복잡하게 얽혀있는 경우가 많기 때문이다. 특히 라이브러리를 물고 있는 Sample의 경우 어떻게 UT를 작성했는지 살펴본다. 

```go
import (
	"bytes"
	"io/ioutil"
	"os"
	"path/filepath"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)
```
assert와 require 패키지를 추가로 사용하였다. 
{% include links.html %}

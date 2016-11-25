---
layout: page
title: xwMOOC 딥러닝
subtitle: 사진속 나이 추정
output:
  html_document: 
    keep_md: yes
  pdf_document:
    latex_engine: xelatex
mainfont: NanumGothic
---

[^visit-south-america]: [박근혜 대통령 중남미 4개국 순방 세부일정](http://www.huffingtonpost.kr/2015/04/15/story_n_7071958.html)


> ## 학습 목표 {.objectives}
>
> * 실제 나이와 사진 속 나이 차이를 수치화한다. 
> * 인공지능 API를 활용하여 사진속 나이를 추정한다.
> * 수많은 사진자료작업을 작업화하는 깨알같은 기법을 학습한다.

## 1. 연령추정 작업흐름 [^pictures-with-age-info]

사진속 연령추정을 위한 작업흐름은 먼저 연령을 추정할 데이터를 구하고 나서, 이를 R로 불러 읽어들인다.
다양한 인공지능 API가 있어 연령추정기능을 제공하는 API를 찾아 이곳에 보낼 수 있는 자료형으로 이미지 데이터를 준비한다.
인공지능 API에 전달하기 전에 이미지 데이터에 대한 기초 분석과 더불어 탐색적 데이터 분석 및 데이터 정제작업을 거친다.

정제된 이미지 데이터가 준비되면 이를 API에 던져 이미지속 성별과 나이결과값을 받아 온다. 그리고 나면 기존 이미지 데이터와 더불어 
애니메이션 제작, 인터랙티브 데이터 분석을 포함한 심도깊은 작업을 수행한다.

<img src="fig/ms-cognitive-age.png" alt="연령 추정 데이터분석 아키텍쳐" width="57%" />


[^pictures-with-age-info]: [2004년부터 2016년까지 촬영된 박 대통령 사진 비교](http://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=100&oid=028&aid=0002342652)

연령추정에 사용될 이미지 데이터는 [2004년부터 2016년까지 촬영된 박 대통령 사진 비교](http://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=100&oid=028&aid=0002342652) 기사에 
올라온 이미지를 다운로드받는다. 다운로드받은 이미지 데이터는 원본데이터로 `03.data/` 같은 폴더를 별로 만들어서 저장하고 안전하게 보관한다. 만약 문제가 발생되게 되면, 다시 
원본 이미지를 분석작업에 활용한다.

## 2. R에서 이미지 기본작업 [^r-magick]

[^r-magick]: [The magick package: Advanced Image-Processing in R](https://cran.r-project.org/web/packages/magick/vignettes/intro.html)

R에서 이미지 분석에 필요한 팩키지를 설치하고 나서 작업환경에 불러온다. 
개별적으로 이미지를 `magick::image_read` 명령어를 통해서 불러오는 것은 사람보다
기계가 더 잘 하는 작업으로 R에서 자주 활용되는 기법을 활용하여 통째로 불러온다.

1. 개별적으로 불러와서 작업한 결과를 담을 통을 준비: `img_info_bucket <- list()`
1. 개별적으로 작업할 이미지 파일 목록을 생성:  `list.files("03_data/")`
1. 반복문을 간결하게 생성: `seq_along()` 함수 사용
1. 반복작업 결과를 집계: `do.call(rbind, img_info_bucket)`

전형적인 [분할-적용-병합(split-apply-combine) 전략](https://statkclee.github.io/r-novice-gapminder/12-plyr-kr.html)의 대표적인 적용사례다.

### 1.1. 이미지 정보 확인

이미지를 불러와서 가장 먼저 할 작업은 데이터프레임을 통해 이미지 데이터 정보를 살펴보는 것이다.
다행히 이미지 파일 형식(**JPEG**)과 넓이와 높이가 모두 동일하고 파일크기만 일부 차이가 
나는 것이 확인된다. 따라서, 별도 전처리 작업은 필요하지 않게 되었다. 기사의 사진 자료를 준비해주신 분께 깊은 감사의 말씀을 전한다.


~~~{.r}
# 0. 환경설정--------------------------------------------------
library(httr)
library(XML)
library(ggplot2)
library(png)
library(grid)
library(jsonlite)
library(dplyr)
library(lubridate)
library(magick)
library(stringr)
library(grid)

# 1. 데이터 불러오기 ----------------------------------------------

img_list <- list.files("03_data/") 
img_list <- stringr::str_replace(img_list, ".jpg", "")

img_info_bucket <- list()

# 2. 사진 정보 확인 ------------------------------------------------
# https://cran.r-project.org/web/packages/magick/vignettes/intro.html
for(lst in seq_along(img_list)){
  tmp <- image_read(paste0("03_data/", img_list[lst], ".jpg"))
  img_info_bucket[[lst]] <- image_info(tmp)
}

img_info_buckets <- do.call(rbind, img_info_bucket)

img_info_df <- data.frame(idate = substr(as.vector(img_list), 5,13), img_info_buckets)

img_info_df
##       idate format width height colorspace filesize
## 1  20040414   JPEG   540    720       sRGB   275964
## 2  20040723   JPEG   540    720       sRGB   241867
## 3  20090702   JPEG   540    720       sRGB   266427
## 4  20110603   JPEG   540    720       sRGB   237164
## 5  20120508   JPEG   540    720       sRGB   261189
## 6  20121108   JPEG   540    720       sRGB   234499
## 7  20121224   JPEG   540    720       sRGB   240137
## 8  20130529   JPEG   540    720       sRGB   277783
## 9  20140113   JPEG   540    720       sRGB   267968
## 10 20140724   JPEG   540    720       sRGB   241137
## 11 20141229   JPEG   540    720       sRGB   183747
## 12 20150504   JPEG   540    720       sRGB   229414
## 13 20150804   JPEG   540    720       sRGB   218613
## 14 20160204   JPEG   540    720       sRGB   238201
## 15 20161104   JPEG   540    720       sRGB   221589

summary(img_info_df)

##    idate              format              width         height     colorspace           filesize     
## Length:15          Length:15          Min.   :540   Min.   :720   Length:15          Min.   :183747  
## Class :character   Class :character   1st Qu.:540   1st Qu.:720   Class :character   1st Qu.:231957  
## Mode  :character   Mode  :character   Median :540   Median :720   Mode  :character   Median :240137  
##                                       Mean   :540   Mean   :720                      Mean   :242380  
##                                       3rd Qu.:540   3rd Qu.:720                      3rd Qu.:263808  
##                                       Max.   :540   Max.   :720                      Max.   :277783  
~~~

### 1.2. 사진 가로로 붙이기

이제 전체 이미지 데이터가 제대로 들어왔는지 다시 한번 시각적으로 확인한다. 개별 파일을 열어서 확인할 수도 있지만,
`image_append` 함수 기능을 사용해서 데이터를 확인한다. 기본설정은 가로방향으로 쭉 붙게 되고, `stack = TRUE` 인자를 넘기면
수직으로 쭉 쌓는 것도 가능하다.


~~~{.r}
# 3. 이미지 쫙 붙이기 ------------------------------------------------
# 전체 이미지 불러오기
for(lst in seq_along(img_list)){
  img_name <- img_list[lst]
  assign(img_name, image_read(paste0("03_data/", img_list[lst], ".jpg")))
}

img_vec <- c(img_20040414, img_20040723, img_20090702, img_20110603, 
             img_20120508, img_20121108, img_20121224, img_20130529, 
             img_20140113, img_20140724, img_20141229, img_20150504,
             img_20150804, img_20160204, img_20161104)


img_left2right <- image_append(image_scale(img_vec, "x77"))

image_write(img_left2right, "04.result/left2right.png", format = "png")

img_top2bottom <- image_append(image_scale(img_vec, "x77"), stack = TRUE)
~~~

<img src="fig/left2right.png" alt="이미지 사진 가로로 쭉 붙이기" width="100%" />


### 1.3. 애니메이션 만들기

이미지를 생성시킨 다음에 가장 나이가 적은 사진과 가장 나이가 많은 사진을 뽑아 
`image_morph()` 기능을 통해 10년이 넘는 동안 얼굴변화과정을 `.gif` 파일로 변환하여 
동적인 시간정보를 담아 낸다.


~~~{.r}
# 이미지 좌우 반전
img_20161104 <- image_flop(img_20161104)

img_transition <- image_morph(c(img_20161104, img_20161104), frames = 10)
img_animation <- image_animate(img_transition, fps=5)

image_write(img_animation, "04.result/img_transition.gif")
~~~

<img src="fig/img_transition.gif" alt="얼굴 변화 애니메이션" width="57%" />












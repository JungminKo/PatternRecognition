## 기하학적 변환
### Upscaling

느낀 점 : 예전에 수업 때 mfc로 해봤기 때문에 어렵지 않을줄 알았으나, 생각보다 고전했던 과제이다.
- 이유 : 메인 코드는 잘 바꿨는데 upscaling하면서 달라진 영상의 가로, 세로를 header에게 전달하지 않았다!!

아쉬운 점 : 맨아래줄과 오른쪽줄의 경우 보간법을 적용하지 못하였다. 

```C
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

#define size 3

void main() {

	BITMAPFILEHEADER hf; // BMP 파일헤더 14Bytes
	BITMAPINFOHEADER hInfo; // BMP 인포에더 40Bytes
	RGBQUAD hRGB[256]; // 팔레트 (256*4 Bytes)

	FILE *fp;
	fp = fopen("lenna.bmp", "rb");
	if (fp == NULL)
	{
		printf("File not found\n");
		return;
	}
	fread(&hf, sizeof(BITMAPFILEHEADER), 1, fp);
	fread(&hInfo, sizeof(BITMAPINFOHEADER), 1, fp);
	fread(hRGB, sizeof(RGBQUAD), 256, fp);

	int W, H, ImgSize;
	W = hInfo.biWidth;
	H = hInfo.biHeight;

	ImgSize = W * H;

	BYTE * Image = (BYTE *)malloc(ImgSize);
	BYTE * Output = (BYTE *)malloc(ImgSize*size*size);

	fread(Image, sizeof(BYTE), ImgSize, fp);
	fclose(fp);

	int out_W = W * size;
	int out_H = H * size;

	// output 영상 image에 맞게 header 정보 바꾸기
	hf.bfSize = (WORD) (out_W * out_H + 1078); 
	hInfo.biWidth = (LONG)out_W;
	hInfo.biHeight = (LONG)out_H;
	hInfo.biSizeImage = (LONG)(out_W * out_H);


	for (int i = 0; i < ImgSize*size*size; i++) {
		Output[i] = 0;
	}

	for (int i = 0; i < out_H-1; i++) {
		float beta = (i % size)/(float)size;

		for (int j = 0; j < out_W-1; j++) {
			float alpha = (j%size) / (float)size;

			int a = i / size;
			int b = j / size;

			BYTE A = Image[a*W+b];
			BYTE B = Image[a*W+b+1];
			BYTE C = Image[(a+1)*W+b];
			BYTE D = Image[(a + 1)*W + b+1];
			
			Output[i*(out_W) + j] = (BYTE)(A*(1 - alpha)*(1 - beta) + B * alpha*(1 - beta) + C * (1 - alpha)*beta + D * alpha*beta);
		}
	}

	fp = fopen("output.bmp", "wb");
	fwrite(&hf, sizeof(BYTE), sizeof(BITMAPFILEHEADER), fp);
	fwrite(&hInfo, sizeof(BYTE), sizeof(BITMAPINFOHEADER), fp);
	fwrite(hRGB, sizeof(RGBQUAD), 256, fp);
	fwrite(Output, sizeof(BYTE), ImgSize*size*size, fp);
	fclose(fp);
	free(Image);
	free(Output);
}
```

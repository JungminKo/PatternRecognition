### Gonzalez&Woods 경계값 자동 결정방법

```C
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

void  main() {

	BITMAPFILEHEADER hf; // BMP 파일헤더 14Bytes
	BITMAPINFOHEADER hInfo; // BMP 인포에더 40Bytes
	RGBQUAD hRGB[256]; // 팔레트 (256*4 Bytes)
	
	FILE *fp;
	fp = fopen("coin.bmp", "rb");
	if (fp==NULL)
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
	BYTE * Output = (BYTE *)malloc(ImgSize);

	fread(Image, sizeof(BYTE), ImgSize, fp);
	fclose(fp);

	BYTE max = 0;
	BYTE min = 255;

	for (int i=0; i < ImgSize; i++) {
		if (Image[i] > max) {
			max = Image[i];
		}
		if (Image[i] < min) {
			min = Image[i];
		}
	}
	BYTE threshold = (BYTE)((max + min) / 2);
	
	int G1, G2, num1, num2;
	double m1, m2;
	BYTE new_threshold;

	while (1) {
		G1 = 0;
		G2 = 0;
		num1 = 0;
		num2 = 0;

		for (int i = 0; i < ImgSize; i++) {
			if (Image[i] > threshold) {
				G1 += Image[i];
				num1++;
			}
			else {
				G2 += Image[i];
				num2++;
			}
		}

		m1 = (double)G1 / num1;
		m2 = (double)G2 / num2;

		new_threshold = (m1 + m2) / 2.0;
		printf("threshold : %d\n", new_threshold);
		if (abs((int)new_threshold - threshold)<2) {
			break;
		}
		threshold = new_threshold;
	}

	for (int i = 0; i <ImgSize ; i++) {
		if (Image[i] > new_threshold) {
			Output[i] = 255;
		}
		else {
			Output[i] = 0;
		}
	}

	fp = fopen("output.bmp", "wb");
	fwrite(&hf, sizeof(BYTE), sizeof(BITMAPFILEHEADER), fp);
	fwrite(&hInfo, sizeof(BYTE), sizeof(BITMAPINFOHEADER), fp);
	fwrite(hRGB, sizeof(RGBQUAD), 256, fp);
	fwrite(Output, sizeof(BYTE), ImgSize, fp);
	fclose(fp);
	free(Image);
	free(Output);
}
```

{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": [],
      "authorship_tag": "ABX9TyPxaJECeykZdvy6yqWaZHxS",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/github/roccaab/WaveletGAN/blob/main/readme.md\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Abstract\n",
        "\n",
        "In questo lavoro proponiamo un’architettura **cGAN** per la generazione di accelerogrammi sintetici di durata 10 secondi, definiti come 5 secondi precedenti e 5 secondi successivi al picco **PGA massimo**. Il segnale, campionato a 200 Hz (2000 campioni totali), viene normalizzato e decomposto in 6 componenti tramite la trasformata wavelet (db4). Ogni componente viene poi valutata da un discriminatore dedicato e la funzione obiettivo finale integra i contributi di ciascun discriminatore con un peso uniforme **α = 1/6**. Il condizionamento (magnitudo e PGA) viene incorporato tramite una semplice trasformazione testo–vettore (_textToVect_) per alimentare il generatore e i discriminatori.\n",
        "\n",
        "# Introduzione\n",
        "\n",
        "La sintesi di accelerogrammi realistici è fondamentale per applicazioni in ingegneria sismica. In questo contesto, una **cGAN condizionata** risulta particolarmente efficace, in quanto permette di generare segnali che rispettino determinate caratteristiche sismiche. Il presente approccio prevede:\n",
        "\n",
        "- **L’estrazione di un segmento di 10 secondi** centrato sul picco PGA.\n",
        "- **La normalizzazione del segnale** (sottrazione della media e divisione per la deviazione standard).\n",
        "- **La decomposizione del segnale** in 6 componenti wavelet (db4).\n",
        "- **L’uso di 6 discriminatori**, ciascuno dedicato a una componente della decomposizione.\n",
        "- **La definizione di una funzione obiettivo** che somma le perdite dei discriminatori pesate uniformemente.\n",
        "\n",
        "# Metodologia\n",
        "\n",
        "## Segmentazione e Normalizzazione\n",
        "\n",
        "Ogni accelerogramma viene segmentato in una finestra di 10 secondi (5 secondi prima e 5 dopo il picco PGA massimo). Con un tasso di campionamento di 200 Hz, ogni segmento è composto da **2000 campioni**. Prima dell’addestramento, il segnale viene normalizzato secondo la seguente formula:\n",
        "\n",
        "\\[\n",
        "x_{\\text{norm}} = \\frac{x - \\mu}{\\sigma}\n",
        "\\]\n",
        "\n",
        "dove **μ** è la media e **σ** la deviazione standard del segnale.\n",
        "\n",
        "## Condizionamento\n",
        "\n",
        "Le informazioni di condizionamento (magnitudo e PGA) sono lette da un file JSON e trasformate in vettori mediante una funzione **textToVect** (in questo esempio, implementata in modo semplificato).\n",
        "\n",
        "## Decomposizione Wavelet\n",
        "\n",
        "Utilizziamo la funzione `pywt.wavedec` con wavelet **db4** per decomporsi il segnale in 6 componenti di dettaglio. Poiché la lunghezza dei coefficienti varia, ciascuna componente viene ricampionata (mediante `pywt.upcoef`) per ottenere un vettore di 2000 campioni, mantenendo così la corrispondenza dimensionale.\n",
        "\n",
        "## Architettura cGAN\n",
        "\n",
        "Il generatore riceve in ingresso:\n",
        "\n",
        "- Un vettore latente \\( z \\in \\mathbb{R}^{100} \\) estratto casualmente.\n",
        "- Il vettore di condizionamento (ad esempio, di dimensione 2: \\([magnitudo, \\, PGA]\\)) ottenuto dalla funzione **textToVect**.\n",
        "\n",
        "Il vettore combinato viene trasformato da uno strato fully connected, riformattato in una matrice per essere elaborata da strati di convoluzione trasposta (ConvTranspose1d) e produrre un accelerogramma sintetico di 2000 campioni.\n",
        "\n",
        "Sono invece utilizzati 6 discriminatori, ognuno dei quali riceve in ingresso una singola componente wavelet (dimensione: 2000 campioni, trattata come un segnale 1D con canale unico) e il vettore di condizionamento. Ciascun discriminatore ha una rete **CNN1D** che estrae feature e, dopo una concatenazione con le informazioni di condizionamento, produce una stima della probabilità che il segnale sia reale.\n",
        "\n",
        "La funzione obiettivo totale combina i contributi dei 6 discriminatori con un coefficiente **α = 1/6** per ciascuno, garantendo che nessun discriminatore domini il training.\n"
      ],
      "metadata": {
        "id": "l_L7kxgEmCds"
      }
    }
  ]
}
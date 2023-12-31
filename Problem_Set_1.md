# -*- coding: utf-8 -*-
"""Problem_Set_1.ipynb

Automatically generated by Colaboratory.

Original file is located at
    https://colab.research.google.com/drive/1SFlf_ZjDVtHJbsu1z0Y40E8y4MzvkXZT
"""

# Importing necessary libraries
import numpy as np
import matplotlib.pyplot as plt
import torch
from torchvision import datasets
from skimage.util import montage
!pip install wandb
import wandb as wb
from skimage.io import imread

# Define a function to convert data to a PyTorch tensor with gradients enabled and place it on a GPU
def GPU(data):
    return torch.tensor(data, requires_grad=True, dtype=torch.float, device=torch.device('cuda'))

# Define a function to convert data to a PyTorch tensor without gradients and place it on a GPU
def GPU_data(data):
    return torch.tensor(data, requires_grad=False, dtype=torch.float, device=torch.device('cuda'))

# Define a function to plot an image
def plot(x):
    if type(x) == torch.Tensor:
        x = x.cpu().detach().numpy()

    fig, ax = plt.subplots()
    im = ax.imshow(x, cmap='gray')
    ax.axis('off')
    fig.set_size_inches(7, 7)
    plt.show()

# Define a function to plot a montage of images
def montage_plot(x):
    x = np.pad(x, pad_width=((0, 0), (1, 1), (1, 1)), mode='constant', constant_values=0)
    plot(montage(x))

# Load MNIST dataset
train_set = datasets.MNIST('./data', train=True, download=True)
test_set = datasets.MNIST('./data', train=False, download=True)

# Extract data and labels from the dataset
X = train_set.data.numpy()
X_test = test_set.data.numpy()
Y = train_set.targets.numpy()
Y_test = test_set.targets.numpy()

# Preprocess image data
X = X[:, None, :, :] / 255
X_test = X_test[:, None, :, :] / 255

# Montage plot a subset of images
montage_plot(X[125:150, 0, :, :])

# Reshape image tensors
X = X.reshape(X.shape[0], 784)
X_test = X_test.reshape(X_test.shape[0], 784)

# Convert data to GPU tensors
X = GPU_data(X)
Y = GPU_data(Y)
X_test = GPU_data(X_test)
Y_test = GPU_data(Y_test)

# Extract a subset of image data
x = X[:, 0:64]

# Transpose the image data
X = X.T

# Create a random model 'M' and perform matrix multiplication
M = GPU(np.random.rand(10, 784))

# Define batch size and extract a batch of image data
batch_size = 64
x = X[:, 0:batch_size]
M = GPU(np.random.rand(10, 784))

# Perform matrix multiplication
y = M @ x

# Calculate accuracy based on model predictions
y = torch.argmax(y, 0)
torch.sum((y == Y[0:batch_size])) / batch_size

# Train a random walk model to achieve at least 75% accuracy
m_best = 0
acc_best = 0

for i in range(100000):
    step = 0.0000000001
    m_random = GPU_data(np.random.randn(10, 784))
    m = m_best + step * m_random

    y = m @ X

    y = torch.argmax(y, axis=0)

    acc = ((y == Y)).sum() / len(Y)

    if acc > acc_best:
        print(acc.item())
        m_best = m
        acc_best = acc

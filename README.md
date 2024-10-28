## AIM
To create a classifier using specialized layers for text data such as Embedding and GlobalAveragePooling1D.

## PROBLEM STATEMENT
The program enables us to classify the given BBC dataset into its respective areas like different categories, for example buisness, sports and tech using Deep learning techniques, which includes loading and preprocessing the data, creating the neural network model, training and evaluation its performance.

## DATASET
![image](https://github.com/user-attachments/assets/325f2f74-2059-480f-a8e7-3c7f6b0bcf12)


## DESIGN STEPS

### STEP 1: 
Unzip the zip file and load the BBC news dataset, split it into training and validation dataset.

### STEP 2: 
Implement a function to convert the text into lower cases, remove the stop words and eliminate punctuation.


### STEP 3: 
Create a TextVectorizer layer to tokenize and convert the dataset into sequences for model training.


### STEP 4: 
Use TensorFlow's StringLookup layer to encode text labels into numerical format.

Write your own steps
### STEP 5: 
Build a model using an embedding layer, global average pooling, and dense layers for multi-class classification

### STEP 6: 
Train the model for 30 epochs using the prepared training data and validate its performance on the validation set.

### STEP 7: 
Evaluate the model's accuracy and loss, and plot the results to track performance over time.
## PROGRAM
### Name:sreeja.v
### Import necessary packages:
```
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
```
### Import zip file
```
#To unzip and read the csv file inside the zip file
import zipfile
with zipfile.ZipFile('/content/BBC News Train.csv.zip', 'r') as zip_ref:
    zip_ref.extractall('extracted_data')
```
```
with open("extracted_data/BBC News Train.csv", 'r') as csvfile:
    print(f"First line (header) looks like this:\n\n{csvfile.readline()}")
    print(f"The second line (first data point) looks like this:\n\n{csvfile.readline()}")
```
### Define the global variables:
```
# Define the global variables
VOCAB_SIZE = 1000
EMBEDDING_DIM = 16
MAX_LENGTH = 120
TRAINING_SPLIT = 0.8
```
### Shape of the data:
```
data_dir = "/content/bbc-tect.csv"
data = np.loadtxt(data_dir, delimiter=',', skiprows=0, dtype='str', comments=None)
print(f"Shape of the data: {data.shape}")
print(f"{data[0]}\n{data[1]}")
```

![image](https://github.com/user-attachments/assets/c1197205-ca53-4bdc-9567-7e55c0ed2321)
### Check for the labels of the data:
```
# Print the first 5 labels from the first column
print(f"The first 5 labels are {data[:5, 0]}")
```

![image](https://github.com/user-attachments/assets/92e03260-31f0-4034-ba17-704934237699)

```
import numpy as np
# Reload the CSV file, but skip the first row which contains the headers
data = np.loadtxt("/content/bbc-tect.csv", delimiter=',', skiprows=1, dtype='str', comments=None)
# Now the dataset will not include the header, and you'll work with the actual data
print(f"There are {len(data)} sentence-label pairs in the dataset.\n")
# Check the number of words in the first sentence (now it should be correct)
print(f"First sentence has {len(data[0, 1].split())} words.\n")
# Print the first 5 labels (category column, index 0)
print(f"The first 5 labels are {data[:5, 0]}")
```

![image](https://github.com/user-attachments/assets/7d47d41c-7a2b-4c79-a8ca-1ca8c2e8ebdc)

### Training and validating the dataset:
```
# GRADED FUNCTIONS: train_val_datasets
def train_val_datasets(data,train_split=1780/2225):
    '''
    Splits data into traning and validations sets
    Args:
        data (np.array): array with two columns, first one is the label, the second is the text
    Returns:
        (tf.data.Dataset, tf.data.Dataset): tuple containing the train and validation datasets
    '''
   ### START CODE HERE ###
    # Compute the number of samples that will be used for training
    train_size = int(len(data) * train_split)
    # Slice the dataset to get only the texts and labels
    texts = data[:, 1]  # texts are in the second column
    labels = data[:, 0]  # labels are in the first column
    # Split the texts and labels into train/validation splits
    train_texts = texts[:train_size]
    validation_texts = texts[train_size:]
    train_labels = labels[:train_size]
    validation_labels = labels[train_size:]
    # Create the train and validation datasets from the splits
    train_dataset = tf.data.Dataset.from_tensor_slices((train_texts, train_labels))
    validation_dataset = tf.data.Dataset.from_tensor_slices((validation_texts, validation_labels))
    ### END CODE HERE ###
    return train_dataset, validation_dataset
# Create the datasets
train_dataset, validation_dataset = train_val_datasets(data)
print('Name: Meetha Prabhu       Register Number: 212222240065      ')
print(f"There are {train_dataset.cardinality()} sentence-label pairs for training.\n")
print(f"There are {validation_dataset.cardinality()} sentence-label pairs for validation.\n")
```
![Screenshot 2024-10-28 092030](https://github.com/user-attachments/assets/6833e1af-d66e-4c84-8621-821a2cb996f8)

### Standardize the Function:
```
def standardize_func(sentence):
    """
    Removes a list of stopwords
    Args:
        sentence (tf.string): sentence to remove the stopwords from
    Returns:
        sentence (tf.string): lowercase sentence without the stopwords
    """
    # List of stopwords
    stopwords = ["a", "about", "above", "after", "again", "against", "all", "am", "an", "and", "any", "are", "as", "at", "be", "because", "been", "before", "being", "below", "between", "both", "but", "by", "could", "did", "do", "does", "doing", "down", "during", "each", "few", "for", "from", "further", "had", "has", "have", "having", "he", "her", "here",  "hers", "herself", "him", "himself", "his", "how",  "i", "if", "in", "into", "is", "it", "its", "itself", "let's", "me", "more", "most", "my", "myself", "nor", "of", "on", "once", "only", "or", "other", "ought", "our", "ours", "ourselves", "out", "over", "own", "same", "she",  "should", "so", "some", "such", "than", "that",  "the", "their", "theirs", "them", "themselves", "then", "there", "these", "they", "this", "those", "through", "to", "too", "under", "until", "up", "very", "was", "we",  "were", "what",  "when", "where", "which", "while", "who", "whom", "why", "why", "with", "would", "you",  "your", "yours", "yourself", "yourselves", "'m",  "'d", "'ll", "'re", "'ve", "'s", "'d"]
    # Sentence converted to lowercase-only
    sentence = tf.strings.lower(sentence)
    # Remove stopwords
    for word in stopwords:
        if word[0] == "'":
            sentence = tf.strings.regex_replace(sentence, rf"{word}\b", "")
        else:
            sentence = tf.strings.regex_replace(sentence, rf"\b{word}\b", "")
    # Remove punctuation
    sentence = tf.strings.regex_replace(sentence, r'[!"#$%&()\*\+,-\./:;<=>?@\[\\\]^_`{|}~\']', "")
    return sentence
```
### Fit-vectorizer Funtion:
```
# GRADED FUNCTION: fit_vectorizer
def fit_vectorizer(train_sentences, standardize_func):
    '''
    Defines and adapts the text vectorizer
    Args:
        train_sentences (tf.data.Dataset): sentences from the train dataset to fit the TextVectorization layer
        standardize_func (FunctionType): function to remove stopwords and punctuation, and lowercase texts.
    Returns:
        TextVectorization: adapted instance of TextVectorization layer
    '''
    ### START CODE HERE ###
    # If train_sentences is a NumPy array, convert it to a TensorFlow Dataset
    if isinstance(train_sentences, np.ndarray):
        train_sentences = tf.data.Dataset.from_tensor_slices(train_sentences)
    # Initialize the TextVectorization layer
    vectorizer = tf.keras.layers.TextVectorization(
        standardize=standardize_func,     # Using the custom standardization function
        max_tokens=1000,                  # Set the vocabulary size to 1000
        output_sequence_length=120        # Set the maximum sequence length to 120
    )
    # Adapt the vectorizer to the training sentences
    vectorizer.adapt(train_sentences.map(lambda text: text))  # Adapting only the text from (text, label) pairs
    ### END CODE HERE ###
### Register Number:
    return vectorizer
```
### Create the vectorizer:
```
Include your code(Model, compile and fit functions) here
# Create the vectorizer
text_only_dataset = train_dataset.map(lambda text, label: text)
vectorizer = fit_vectorizer(text_only_dataset, standardize_func)
vocab_size = vectorizer.vocabulary_size()
print('Name: sreeja.v       Register Number: 212222230169      ')
print(f"Vocabulary contains {vocab_size} words\n")
```

## OUTPUT
![Screenshot 2024-10-28 092118](https://github.com/user-attachments/assets/4dade343-a3f2-47b8-ba35-ec5ba535308a)


### Label encoder Function:
```
# GRADED FUNCTION: fit_label_encoder
def fit_label_encoder(train_labels, validation_labels):
    """Creates an instance of a StringLookup, and trains it on all labels
    Args:
        train_labels (tf.data.Dataset): dataset of train labels
        validation_labels (tf.data.Dataset): dataset of validation labels
    Returns:
        tf.keras.layers.StringLookup: adapted encoder for train and validation labels
    """
    ### START CODE HERE ###
    # Join the two label datasets by concatenating them
    labels = train_labels.concatenate(validation_labels)
    # Instantiate the StringLookup layer. We set mask_token=None and num_oov_indices=0 to avoid OOV tokens
    label_encoder = tf.keras.layers.StringLookup(mask_token=None, num_oov_indices=0)
    # Fit the StringLookup layer on the concatenated labels
    label_encoder.adapt(labels)
    ### END CODE HERE ###
    return label_encoder
```
### Create the label Encoder:
```
# Create the label encoder
train_labels_only = train_dataset.map(lambda text, label: label)
validation_labels_only = validation_dataset.map(lambda text, label: label)
label_encoder = fit_label_encoder(train_labels_only,validation_labels_only)
print('Name: sreeja.v      Register Number: 21222223-169      ')
print(f'Unique labels: {label_encoder.get_vocabulary()}')
```

![Screenshot 2024-10-28 092212](https://github.com/user-attachments/assets/d9196ca1-cfb6-4beb-8492-27f35126d82b)

### Preprocess the data function:
```
# GRADED FUNCTION: preprocess_dataset
def preprocess_dataset(dataset, text_vectorizer, label_encoder):
    """Apply the preprocessing to a dataset
    Args:
        dataset (tf.data.Dataset): dataset to preprocess
        text_vectorizer (tf.keras.layers.TextVectorization ): text vectorizer
        label_encoder (tf.keras.layers.StringLookup): label encoder
    Returns:
        tf.data.Dataset: transformed dataset
    """
      ### START CODE HERE ###
    # Apply text vectorization and label encoding
    dataset = dataset.map(lambda text, label: (text_vectorizer(text), label_encoder(label)))
    # Set the batch size to 32
    dataset = dataset.batch(32)
    ### END CODE HERE ###
    return dataset
# Preprocess your dataset
train_proc_dataset = preprocess_dataset(train_dataset, vectorizer, label_encoder)
validation_proc_dataset = preprocess_dataset(validation_dataset, vectorizer, label_encoder)
train_batch = next(train_proc_dataset.as_numpy_iterator())
validation_batch = next(validation_proc_dataset.as_numpy_iterator())
print('Name: sreeja      Register Number: 212222230169      ')
print(f"Shape of the train batch: {train_batch[0].shape}")
print(f"Shape of the validation batch: {validation_batch[0].shape}")
```


![Screenshot 2024-10-28 092320](https://github.com/user-attachments/assets/297a4b53-7a9c-4c5a-983e-11661a592d55)

### Create Model:
```
# GRADED FUNCTION: create_model
def create_model():
    """
    Creates a text classifier model
    Returns:
      tf.keras Model: the text classifier model
    """
      ### START CODE HERE ###
    model = tf.keras.Sequential([
        tf.keras.Input(shape=(120,)),  # Input layer with a fixed sequence length of 120
        tf.keras.layers.Embedding(input_dim=1000, output_dim=16),  # Smaller embedding layer (reduce output_dim)
        tf.keras.layers.GlobalAveragePooling1D(),  # Global average pooling to reduce complexity
        tf.keras.layers.Dense(16, activation='relu'),  # Reduced Dense layer size to 16 units
        tf.keras.layers.Dense(5, activation='softmax')  # Output layer with 5 units for 5 classes
    ])
    # Compile the model with appropriate loss, optimizer, and metrics
    model.compile(
        loss='sparse_categorical_crossentropy',  # Use sparse categorical cross-entropy for integer-encoded labels
        optimizer='adam',  # Adam optimizer
        metrics=['accuracy']  # Track accuracy
    )
    ### END CODE HERE ###
    return model
# Get the untrained model
model = create_model()
```
### Model Evaluation:
```
example_batch = train_proc_dataset.take(1)
try:
	model.evaluate(example_batch, verbose=False)
except:
	print("Your model is not compatible with the dataset you defined earlier. Check that the loss function and last layer are compatible with one another.")
else:
	predictions = model.predict(example_batch, verbose=False)
	print(f"predictions have shape: {predictions.shape}")
```

![image](https://github.com/user-attachments/assets/fe557196-1c0f-4085-9e3e-cdc7c02870f5)

### Fit the model:
```
history = model.fit(train_proc_dataset, epochs=30, validation_data=validation_proc_dataset)
```

### Plot the graph (function):
```
def plot_graphs(history, metric):
    plt.plot(history.history[metric])
    plt.plot(history.history[f'val_{metric}'])
    plt.xlabel("Epochs")
    plt.ylabel(metric)
    plt.legend([metric, f'val_{metric}'])
    plt.show()
print('Name:sreeja       Register Number: 212222230169      ')
plot_graphs(history, "accuracy")
plot_graphs(history, "loss")
```
### Name: sreeja

### Register Number: 212222230169

## OUTPUT:
### Loss, Accuracy Vs Iteration Plot

![image](https://github.com/user-attachments/assets/83cfaa22-f0c8-46df-8cf1-b33c24d6abaf)

![image](https://github.com/user-attachments/assets/d320d647-ec5e-46eb-b33f-a4c483c68656)




## RESULT
Thus the program to create a classifier using specialized layers for text data such as Embedding and GlobalAveragePooling1D is implemented successfully.

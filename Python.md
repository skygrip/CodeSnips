# Python Code Snips

Select a filename with a GUI element

    from tkinter.filedialog import askopenfilename
    filename = askopenfilename(initialdir = sys.path[0])

Read a CSV file into a panda dataframe

    filename = str(input("Enter filename to read results from: "))
    print(f"Filename to read results from: {filename}")

    if os.path.isfile(filename) == False:
        raise Exception(f"Error: File {filename} not found")

    urls = pandas.read_csv(filename)

Loop over a Panda rows with iterrows. Note iterrows is slow
    df = pd.DataFrame([[4, 9]] * 3, columns=['A', 'B'])

    for i, x in df.iterrows():

        # optionally change a value
        df.at[i,'B'] = 100

Loop over a panda with lambdas (faster)

    df = pd.DataFrame([[4, 9]] * 3, columns=['A', 'B'])
    df['B'] = df['B'].apply(lambda x: 1000)

Loop over a panda with lambdas and create a copy

    df = pd.DataFrame([[4, 9]] * 3, columns=['A', 'B'])
    # Assign can create new columns as well as overwriting existing ones
    df = df.assign(B=lambda x: 100, C=lambda x: x['A']+x['B'])

    #same thing but with functions
    def func(x):
        return x['A'] + x['B']

    df = df.assign(B=lambda x: 100, C=lambda x: func(x))
    





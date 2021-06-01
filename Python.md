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


# Novel tools

[中文链接](/README_CN.md)

## Why am I doing this

First I want to affirm that I do not support piracy. Please purchase/subscribe to the novels on their official websites/apps if possible. My tools are mainly used on novels that I subscribed.

However, websites like Qidian would censor some/all chapters of a novel, which can be quite frustrating when you are midway through a novel. When this happens, one has to rely on text versions from the Internet. Plain text files have no format, and if you want a quick read, that would be fine. However, once you want to compile it into a more e-reader friendly format (like epub), problems arise. While some apps could auto match chapter names and generate a table of contents, some questions linger:

- No support for custom/irregular volume/chapter titles;

  - Some novels might use unorthodox titles, like "Songs of the North, Part 1".

- Potential duplicate or missing chapter indices;

  - The author might make a mistake and accidentally put two Chapter 10's, or they may skip Chapter 11 and advance to Chapter 12 after Chapter 10.

- Varying formats of titles.

  - The author might mix "Chapter 1: The Uprising" and "Chapter 1. The Uprising".

This would result in a table of content that is incomplete and full of errors -- also not good to the reading experience. This toolkit tries to solve this problem by providing a unified framework to analyze novel files, generate a better table of contents, and reformat the novel. All novels have their unique problems, and it is unlikely that a universal script would suit every need. However, at the very least, one can generate an initial novel structure, and manually correct it if needed.

## Installation

This project is managed by [uv](https://docs.astral.sh/uv/), and is built on Python 3.10. After installing uv, run `uv sync --dev` to install all dependencies. Run `uv run python -m pytest` to run all the tests.

If you simply want to use the toolkit without writing your extensions, run `uv sync` (without `--dev`).

If you don't have uv, you will need to install the dependencies manually. If you would like to use `DirectoryReader` or `DirectoryWriter`, you would need `natsort`, which is used to sort the numbers in natural order (1, 2, ... 10, 11 instead of 1, 10, 11, ... 2):

```shell
pip3 install natsort
```

If you would like to use `EpubWriter`, you would need `markdown`, `ebooklib` and `bs4`:

```shell
pip3 install markdown ebooklib bs4
```

## Basic Concepts

- A novel, or any book, usually consists of **volumes** and **chapters**. Some novels may only have chapters, but not volumes.
- Volumes and chapters are denoted by **titles** -- **volume titles** and **chapter titles**, respectively.
- The book might have a **book introduction** at be beginning of the text file. It may also contain **volume introductions** at the beginning of each volume.
- Therefore, for a book, the common structure is: **book title** - book introduction - volume title - volume introduction - chapter title - **chapter content** - ...
- **Regular titles** contain an **index** and a **content**.  Sometimes they may not have content at all, but an index is always necessary. For example, *Volume 1: The Escape* has the index *1* and content *The Escape*.
- There might be several sets of indices within a same book. For example, there might be some *Interlude* chapters scattered across normal chapters. These interludes are indexed independently of the others.
- **Special titles** do not contain an index. Like regular titles, they may or may not have content, but they may contain certain **affixes** that make them easy to recognize. Examples include *Introduction*, *Foreword*, and *Conclusion*.
- **Metadata** are data about the book that are not part of the content itself, including the book author, id, languages, and tags. They are useful for organizing books in an e-book management software, including [Calibre](https://calibre-ebook.com/). A template metadata can be found [here](config/sample_metadata.json).

The job of this toolkit is to extract the different elements within a novel file, correct them of any errors (if any), and reformat them to be more regular and nicer.

## Usage

If you are using uv, use `uv run python -m ntcli -h` for the list of supported commands and arguments.

After you download the novel, remove all the ads first, if there are any. Typically, there are two workflows:

1. Create a "structure file" that describes how to locate the volume and chapter titles in the text file, and how to format these titles. (**struct**)
2. Manually inspect the structure file for any errors, and modify them as needed. If necessary, you can modify the novel text as well, but make sure to either re-generate a new structure when you are done, or change the existing structure file to match the text.
3. Use the structure file and the source text, reconstruct a formatted file. (**create**)

You can also do the following:

1. Split the source file into individual volume directories, and chapter files within these directories. (**split**)
2. Manually inspect the individual volumes and chapters for any errors. This avoids the problem of editing a large text file, which many text editors have trouble reading
3. Recreate a "structure file" from the volume and chapter files. (**struct_dir**)
4. Manually inspect the structure file for any inconsistencies, and modify them as needed. If a book contains special volumes/chapters, the order of them might be lost during splitting (as they are not indexed). In this case, you might need to rearrange the order of such titles.
5. Use the structure file and the files, reconstruct a formatted file. (**create_dir**)

An optional step is to create a metadata json file for your book, and include some of these metadata within the book.

After you get a formatted book, you may want to add the file to your favorite e-book management software, like [Calibre](https://calibre-ebook.com/), and convert it into some e-book friendly format (aka epub). (**add** and **convert**) 

For the above two tools to work, you will need to metadata file, as well as a book cover named as `cover.jpg`.

## Samples and Documentation

To see what inputs the toolkit can handle and what outputs it can produce, take a look at the [integration tests](/tests/toolkit). To create your own toolkit configurations for your own novel, you may refer to the configs of these tests, and the [default configs](/config). For references on all built-in classes, refer to [References](/docs/references.md). 

The toolkit is very extensible, and you may want to write your own components for your own needs. To understand how the project is structured and what each part does, refer to [Design](/docs/design.md).


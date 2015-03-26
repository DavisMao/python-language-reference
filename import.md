#5.导入系统  
Python 中，一个模块的代码通过[导入](https://docs.python.org/3/glossary.html#term-importing)程序访问另一个[模块](https://docs.python.org/3/glossary.html#term-module)的代码。[导入](https://docs.python.org/3/reference/simple_stmts.html#import)语句是调用导入机制的最常用的方法，但不是唯一方式。像函数 [importlib.import_module()]() 和内建函数 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 也能用来调用导入机制。  

[导入](https://docs.python.org/3/reference/simple_stmts.html#import) 语句包含两个操作：首先查找指定的模块，然后将查找结果绑定到局部作用域内的一个名字上。[导入](https://docs.python.org/3/reference/simple_stmts.html#import)语句中的查找操作被定义为一个具有适当参数的 [_ _import_ _()](https://docs.python.org/3/library/functions.html#__import__) 函数的调用。 [_ _import_ _()](https://docs.python.org/3/library/functions.html#__import__) 的返回值被用作执行导入语句的名称绑定操作。名称绑定操作的准确细节请见 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 语句。  

直接调用 [_ _import_ _()](https://docs.python.org/3/library/functions.html#__import__) 仅执行模块搜索和模块创建操作（如果查找到的话）。虽然可能发生某些附带后果，比如导入父包以及更新不同的缓存（包括 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules)），但只有[导入](https://docs.python.org/3/reference/simple_stmts.html#import)语句执行名称绑定操作。  

当调用 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 作为导入语句的一部分，导入系统首先在模块的全域命名空间中查找指定名称的函数。假如没有找到，则调用标准内建 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 。调用导入系统的其他机制，例如 [importlib.import_module()](https://docs.python.org/3/library/importlib.html#importlib.import_module)，不执行该项核对，并将一直使用标准导入系统。  

当一个模块第一次被导入，Python 搜索该模块，如果找到，它便创建一个模块对象[[1]](https://docs.python.org/3/reference/import.html#fnmo)，并初始化。假如不能找到指定的模块，将引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常 。当调用导入机制时，Python 实现不同的策略去搜索指定的模块。通过使用不同的钩子程序能够修改和扩展这些策略，钩子程序将在下面部分阐述。  

*3.3版本中的变化：*导入系统已更新到完全能够实现 [PEP 302](http://www.python.org/dev/peps/pep-0302)的第二阶段。不再有隐含的导入机制——通过 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path)，整个导入系统是暴露的。此外，已实现本地命名空间包支持（见[PEP 420](http://www.python.org/dev/peps/pep-0420)）。  
##5.1. 导入库  
[importlib](https://docs.python.org/3/library/importlib.html#module-importlib)  模块提供丰富的 API 用以与导入系统交互。例如，为调用导入机制， [importlib.import_module()](https://docs.python.org/3/library/importlib.html#importlib.import_module) 提供了一个推荐的，比内建函数 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 更简单的API。参阅 [importlib](https://docs.python.org/3/library/importlib.html#module-importlib) 库文件了解更多细节。  
##5.2. 包  
无论模块是在 Python、C或是其他语言中实现，Python 只有一个模块对象型态，而且所有模块都是这个型态。为了帮助组织模块以及提供一个命名体系，Python 提供了一个[包](https://docs.python.org/3/glossary.html#term-package)的概念。  

你可以将包当成一个文件系统的目录，将模块当成目录中的文件，但不能太随便地做这样的类比，因为包和模块不需要来自文件系统。为了本文件的目的，我们将使用目录和文件的这个方便的类比。类似文件系统目录，包被分级组织起来，而且包本身也可以包含子包，常规模块也是如此。  

重要的是要牢记所有的包都是模块，但不是所有的模块都是包。或者换一种说法，包仅是一种特殊的模块。具体地说，任何包含 `__path__` 属性的模块被认为是包。  

所有的模块都有一个名称。类似于 Python 标准属性访问语法，子包与他们父包的名字之间用点隔开。因此，你可能有一个称为 [sys](https://docs.python.org/3/library/sys.html#module-sys) 的模块和一个称为 [email](https://docs.python.org/3/library/email.html#module-email) 的包，相应地你可能有一个称为 [email.mime](https://docs.python.org/3/library/email.mime.html#module-email.mime) 的子包和该子包中一个称为 `email.mime.text` 的模块。
###5.2.1. 常规包  
Python 定义两种型态包，[常规包](https://docs.python.org/3/glossary.html#term-regular-package)和[命名空间包](https://docs.python.org/3/glossary.html#term-namespace-package)。常规包是存在于 Python 3.2 及更早版本中的传统包。常规包通常被当作含 `__init__.py` 文件的目录来实现。当导入一个常规包时，该 `__init__.py` 文件被隐式执行，而且它定义的对象被绑定到包命名空间中的名称。 `__init__.py` 文件能包含其他任何模块能够包含的相同的 Python 代码，而且在导入它时，Python 将给模块增加一些额外的属性。  

举个例子，下面这个文件系统布局定义了一个有三个子包的顶层 `parent` 包：  
  
    parent/
        __init__.py
        one/
            __init__.py
        two/
            __init__.py
        three/
            __init__.py  

导入 `parent.one` 将隐式执行 `parent/__init__.py` 和 `parent/one/__init__.py`。 随后导入 `parent.two` 或者 `parent.three` 将分别执行 `parent/two/__init__.py` 和 `parent/three/__init__.py`。  

###5.2.2.命名空间包  
命名空间包是不同[文件集](https://docs.python.org/3/glossary.html#term-portion)的复合，每个文件集给父包贡献一个子包。文件集可以存于文件系统的不同位置。导入过程中 Python 搜索的压缩文件，网络或者其他地方也可以找到文件集。命名空间包可以也可以不与文件系统的对象直接对应。他们可以是真实的模块但没有具体的表述。  

命名空间包不使用寻常列表作为他们 `__path__` 的属性。相反，他们使用自定义迭代器型态，如果他们父包的路径（或者高阶包的 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) ）改变，它将在下次试图导入时在该包中自动重新搜索包部分。  

没有带有命名空间包的 `parent/__init__.py` 文件。实际上，导入搜索中可能有多种 `parent` 目录存在，而每一个目录都被不同的部分提供。因此，`parent/one` 在物理上可能不是紧挨着 `parent/two` 边。在这种情况下，无论在什么时候导入该父包或它的子包时，Python 将为高级别的父包创建一个命名空间包。  

命名空间包的详细说明也参见 [PEP 420](http://www.python.org/dev/peps/pep-0420) 。
##5.3.搜索  

搜索之前，Python 需要导入模块的[全限定名](https://docs.python.org/3/glossary.html#term-qualified-name)（或者包名称，但为了讨论的目的，两者之间的区别并不重要）。该名称可以来自[导入](https://docs.python.org/3/reference/simple_stmts.html#import)语句的不同参数，或者，来自 [importlib.import_module()](https://docs.python.org/3/library/importlib.html#importlib.import_module) 或 [__import__()](https://docs.python.org/3/library/functions.html#__import__) 函数的参数。  

该名称将被用在导入搜索的不同阶段，并且其可以是一个子模块的虚线路径，比如 `foo.bar.baz.`。在这种情况中，Python 首先试图导入 `foo`，然后是 `foo.bar`，最后导入 `foo.bar.baz`。如果中间的任何导入失败，将引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。  

###5.3.1. 模块缓存
进行搜索时，搜索的第一个地方是 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules)。这个映射作为前期已导入的所有模块的一个缓存，包括中间路径。所以，假如 `foo.bar.baz` 前期已被导入，那么，[sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 将包含进入 `foo`，`foo.bar` 和 `foo.bar.baz`的入口。每个键都有自己的数值，都有对应的模块对象。  

导入过程中，在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中查找模块名称，如果存在，相关的值是满足导入的模块，那么程序完成。然而，如果值为 `None`，则引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。如果未找到模块名称，Python 将继续搜索模块。  

[sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 是可写的。删除一个键可能不会毁坏相关模块（因为其他模块可以保持引用它），但是它将使指定模块的缓存入口无效，导致 Python 在下次导入时重新搜索指定的模块。也可以分配键值为 `None`，迫使下一次模块导入时引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。  

注意，假如你保持引用模块对象，并使 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules)中缓存入口无效，然后再重新导入指定的模块，则这两个模块对象将不相同。对比之下， [imp.reload()](https://docs.python.org/3/library/imp.html#imp.reload)将重新使用相同的模块对象，并通过重新运行模块代码简单地重新初始化模块内容。  

###5.3.2. 查找器和加载器
如果指定模块没在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中找到，将调用 Python 的导入协议来查找和加载模块。这个协议包含两个概念性的对象，[查找器](https://docs.python.org/3/glossary.html#term-finder) 和 [加载器](https://docs.python.org/3/glossary.html#term-loader)。一个查找器的任务是决定它是否能够通过运用其所知的任何策略找到指定模块。实现这两个接口的对象称为 [导入器](https://docs.python.org/3/glossary.html#term-importer)。当他们发现他们能够加载所需的模块，他们返回自身。  

Python 包括许多默认的查找器和导入器。第一个知道如何定位内置模块，第二个知道如何定位冻结模块。第三个默认查找器搜索模块的[导入路径](https://docs.python.org/3/glossary.html#term-import-path)。[导入路径](https://docs.python.org/3/glossary.html#term-import-path)是一个位置的列表，这些位置可以命名文件系统路径或者压缩文件。它也可扩展到搜索任何可定位的资源，例如被 URLs 识别的资源。  

导入机制是可扩展的，因此新的查找器能够被添加来扩展模块搜索的范围和广度。  

事实上，查找器不真正加载模块。如果他们能够找到指定的模块，他们返回一个模块分支，即模块导入相关信息的封装，在加载模块时导入机制运用的信息。  

以下部分将更具体讲述查找器 和 加载器的协议，包括如何创建和注册新的协议。  
*版本3.4中的变化：*在 Python 以前的版本中，查找器直接返回[加载器](https://docs.python.org/3/glossary.html#term-loader)，然而现在他们返回含有加载器的模块分支。加载器在导入中仍被使用，但几乎没有责任。  
###5.3.3.导入钩子程序  
导入机制被设计为可扩展的；其基础的运行机制是导入钩子程序。存在两种导入钩子程序的型态：元钩子程序和导入路径钩子程序。  

在其他任何导入程序运行之前，除了 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 缓存查找，在导入处理开始时调用元钩子程序。这就允许元钩子程序覆盖 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) 处理程序，冻结模块，或甚至内建模块。如下所述，通过给 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 添加新的查找器对象注册元钩子程序。  

当他们的相关路径项被冲突时，导入路径钩子程序作为 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) (或者 `package.__path__`)处理程序的一部分被调用。如下所述，通过给 [sys.path_hooks](https://docs.python.org/3/library/sys.html#sys.path_hooks) 添加新的调用来注册导入路径钩子程序。  

###5.3.4. 元路径  
在[sys.modules](https://docs.python.org/3/library/sys.html#sys.modules)中没有找到指定模块时，Python 紧接着会搜索包含一个元路径查找器对象的列表的[sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path)。为了查看他们是否知道如何处理指定模块，这些查找器将被质疑。元路径查找器必须实现 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec) 方法，该函数需要三个参数：名称，导入路径以及（可选）对象模块。元路径查找器能够使用任何想要使用的策略来确定是否它能够处理指定的模块。  

如果元路径查找器知道如何处理指定模块，它就返回一个分支对象。如果不能处理，就返回 `None`。假如 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 处理直到其列表最后也没有返回一个分支，则引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。仅仅传播引起的其他任何异常情况将中止导入处理程序。  

调用元路径查找器的 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec) 方法需要两个或三个参数。第一个是导入模块的全限定名，比如 `foo.bar.baz`。 第二参数是用来模块搜索的路径入口。对于高阶模块来说，第二个参数是 `None`,但是对子模块或者子包来说，第二参数是父包  `__path__` 属性值。如果不能访问准确的 `__path__` 属性，将引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。第三个参数是一个已存在的对象，其随后将是加载的目标。导入系统仅在重新加载时通过一个目标模块。  

一个单一的导入要求可以多次遍历元路径。举例说明，假设还没有缓存任何所涉的模块，在每个元路径查找器(`mpf`)上调用 `mpf.find_spec("foo", None, None)`，导入 `foo.bar.baz` 将首先执行一个高阶导入。导入 `foo` 后，调用 `mpf.find_spec("foo.bar", foo.__path__, None)`，通过再一次遍历元路径，`foo.bar` 将被导入。一旦完成导入 `foo.bar`，最后的遍历将调用 `mpf.find_spec("foo.bar.baz", foo.bar.__path__, None)`。

一些元路径查找器仅支持高阶导入。当除 `None` 以外的的任何东西作为第二参数通过时，这些导入器将一直返回 `None`。  

Python 默认的 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 有三个元路径查找器，一个知道如何导入内建模块，一个知道如何导入冻结模块，另一个知道如何从一个[导入路径](https://docs.python.org/3/glossary.html#term-import-path)(即[路径查找器](https://docs.python.org/3/glossary.html#term-path-based-finder))中导入模块。  

*版本3.4中的变化：*元路径查找器的 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec) 方法取代了现已不被推荐使用的 [find_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_module)。尽管它将继续无变化地运行，但仅当查找器不执行 `find_spec()` 时，导入机制才将试图实现它。  
##5.4. 加载过程  
当找到一个模块分支时，在加载模块时导入机将使用它（及它包含的加载器）。下面是导入加载部分时所发生的近似值：  
    
    module = None  
    if spec.loader is not None and hasattr(spec.loader, 'create_module'):
        module = spec.loader.create_module(spec)
    if module is None:
        module = ModuleType(spec.name)
    # The import-related module attributes get set here:
    _init_module_attrs(spec, module)
    
    if spec.loader is None:
        if spec.submodule_search_locations is not None:
            # namespace package
            sys.modules[spec.name] = module
        else:
            # unsupported
            raise ImportError
    elif not hasattr(spec.loader, 'exec_module'):
        module = spec.loader.load_module(spec.name)
        # Set __loader__ and __package__ if missing.
    else:
        sys.modules[spec.name] = module
        try:
            spec.loader.exec_module(module)
        except BaseException:
            try:
                del sys.modules[spec.name]
            except KeyError:
                pass
            raise
    return sys.modules[spec.name]  

注意以下细节：  

- 如果在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中存在一个指定名称的模块对象，导入将已返回它。  
- 在加载器执行模块代码前，模块存在在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中。这是重要的，因为模块代码可以自我导入（直接或间接地）；预先把它添加到 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中能阻止最坏情况下的无限递推和最好情况下的多重加载。  
- 假如加载失败了，这个失败的模块（仅是这个失败的模块）将从 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 里移除。任何已在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 缓存中的模块，以及任何附带成功加载的模块，都一定留存在缓存之中。这与重新加载时失败模块也能存留在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中的情况相反。  
- 如[后面部分](https://docs.python.org/3/reference/import.html#import-mod-attrs)总结的，创建模块后但在执行前，导入机制设置导入相关模块的属性（以上伪代码案例中的 “_init_module_attrs”）。  
- 模块运行是模块命名空间密集加载的关键时刻。运行完全交给了能够决定什么密集以及怎样密集的加载器。  
- 加载期间创建的和通过 exec_module() 的模块可能不是导入结束时返回的那个[[2](https://docs.python.org/3/reference/import.html#fnlo)]。  
*3.4版本中的变化：*导入系统接替了加载器的公式化任务。这些以前由 [importlib.abc.Loader.load_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.load_module) 方法执行。  

###5.4.1. 加载器
模块加载器提供加载的判定函数：模块执行。导入机制用要执行的模块对象的单一参数调用 [importlib.abc.Loader.exec_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module) 方法。任何从 [exec_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module)返回的值将被忽略。  
加载器必须满足下列要求：  
- 假设模块是一个 Python 模块（与内建模块或动态加载扩展相反），加载器应该在模块全域命名空间（`module.__dict__`）中执行该模块代码。  
- 假如加载器不能执行模块，它应该引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常，即使将传播 [exec_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module) 引发的任何其他异常情况。  

在许多例子中，查找器和加载器可以是相同的对象；在这种情况中，[find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec) 方法将仅返回一个加载器设置为 `self` 的分支。  

模块加载器可以选择在加载时通过实现一个 [create_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.create_module) 方法来创建模块对象。它需要一个参数，即模块分支，并在加载中返回新的模块对象用以使用。`create_module()` 不需要在模块对象上设置任何属性。假如加载器不定义 `create_module()`，导入机制本身将创建新的模块。  

*3.4版本中的更新：*加载器的 `create_module()` 方法。  

*3.4版本中的变化：*[load_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.load_module) 方法被 [exec_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module) 取代，并且导入机制承担了所有加载的公式化任务。  

为了与存在的加载器相兼容，导入机制将使用加载器的 `load_module` 方法（如果存在，并且加载器也不实现 `exec_module()` 的话）。然而， `load_module()` 已不被推崇，加载器应替代实现 `exec_module()`。  
  
除了运行模块之外，`load_module()`方法必须实现以上所述的所有的公式化加载功能。经过一些补充说明，所有相同的约束条件都可应用。  

- 假设在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中存在一个命名的模块对象，加载器必须使用这个存在的模块。（否则，[importlib.reload()](https://docs.python.org/3/library/importlib.html#importlib.reload)不能正确运行）假如在 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules)中不存在指定模块，加载器必须创建一个新的模块对象并把它添加到 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中。  
- 在加载器运行模块代码前，[sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中必须存在模块，以阻止无限递推或多重加载。    
- 假如加载失败，加载器必须移除其嵌入 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中的所有模块。但仅当加载器本身已显示加载过它时，它必须只移除失败模块。  

###5.4.2. 模块分支  

导入机制在导入中，尤其在加载前，使用有关每个模块的多种信息。大部分信息对所有模块都是共用的。模块分支的目的是在每个模块基础上封装导入相关信息。  
 
在导入时使用一个分支允许语句在导入系统组件间传输，例如，在创建模块分支的查找器和运行它的加载器之间传输。最重要的是，它允许导入机制执行加载的公式化操作，而如果没有模块分支，加载器承担那项任务。  

有关模块分支能拥有的相关信息，更详细地参见 [ModuleSpec](https://docs.python.org/3/library/importlib.html#importlib.machinery.ModuleSpec)。  

*3.4版本的更新.*
###5.4.3. 导入关联模块属性  
  
在加载器运行模块前，根据模块分支，导入机制在加载中在每个模块上填入下列属性。  

__name__  

`__name__`属性必须设置为模块的全限定名。该名称用来唯一识别导入系统的模块。  
__loader__  
 
`__loader__`属性必须设置为加载模块时导入机制使用的加载器对象。这主要是为了自我检查，但也可以用作加载器专用的附加功能，例如获得一个加载器相关的数据。  
`__package__`  

模块的`__package__`属性必须设置。它的值必须是一个字符串，但可以与其`__name__`是相同的值。当模块是包时，其`__package__`值应该设定为它的`_name__`。当模块不是包时，对于高阶模块`__package__`应该设置为空字符串，对于子模块，应设置为父包名称。具体详见 [PEP 366](http://www.python.org/dev/peps/pep-0366)。  
 
该属性可以替代 `__name__` 用作计算主要模块的显示相关导入，如 [PEP 366](http://www.python.org/dev/peps/pep-0366) 所定义的。  

__spec__

`__spec__`属性必须设置为导入模块时所使用的模块分支。这主要用作自查以及主要用在加载中。准确设置 `__spec__` 同样应用于解释器启动中初始化的模块。有一个例外是 `__main__`，在一些例子中在 `__spec__` 设置为 `None`。  

*3.4版本的更新：*  

__path__  

假如模块是个包（不管常规包还是命名空间包），该模块对象的 `__path__` 属性必须设置。值必须是迭代的，但如果 `__path__` 没有更深的意义，值也可以是空的。如果 `__path__` 不是空的，迭代结束时它必须产生字符串。更多有关 `__path__` 语义学的细节将在下面给出。
无包模块不应有 `__path__` 属性。  

__file__  
__cached__  

`__file__` 是可选的。如果设置，该属性的值必须是一个字符串。如果没有语义学意义（例如从数据库中加载的一个模块）,导入系统可以选择不设 `__file__`。

如果设置了 `__file__`，也可以适当设置 `__cached__` 属性，该属性是任何代码编译版本的路径（例如字节编译文件）。设置该属性不需要文件存在；路径能够简单地指向编译文件存在的位置。（见 [PEP 3147](http://www.python.org/dev/peps/pep-3147)）。  

没有设置 `__file__` 时，也可以设置　`__cached__` 。然而，那种情况相当非典型。最终，加载器是利用 `__file__` 和/或 `__cached__` 的使用者。所以假如一个加载器能够从一个缓存模块中加载而不能从一个文件中加载的话，适用这个非典型的情况可能是合适的。
###5.4.4. module.__path__   

根据定义，如果一个模块有一个 `__path__` 属性，无论它的值是什么，它就是一个包。  

一个包的 `__path__` 属性在导入其子包时使用。在导入机制中，它与 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) 的功能很相同，即在导入时提供一个搜索模块的位置列表。然而，通常 `__path__` 比 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) 限制更多。  

`__path__` 必须是字符串的一个迭代，但它可以为空。[sys.path](https://docs.python.org/3/library/sys.html#sys.path) 中使用的相同的规则也适用包的 `__path__`，并且当遍历包路径时，将求助[sys.path_hooks](https://docs.python.org/3/library/sys.html#sys.path_hooks) 下面将讲到）。  

一个包的 `__init__.py` 文件可以设置或改变该包的 `__path__` 属性，并且这也是 [PEP 420](http://www.python.org/dev/peps/pep-0420) 之前实现命名空间包的典型方式。采用 [PEP 420](http://www.python.org/dev/peps/pep-0420)之后，命名空间包不再需要提供只包含 `__path__` 操作代码的 `__init__.py` 的文件；导入机制自动准确设置命名空间包的 `__path__`。  
###5.4.5.模块 reprs

通过默认，所有模块都有一个可用的表示，然而，根据上述所设的各属性，在模块分支中，你可以更明确地控制各模块对象的表示。  

假如模块存在一个分支 (`__spec__`)，导入机制将试图从它那生成一个表示。如果失败或不存在分支，导入系统将使用模块上任何可用信息制作一个默认的表示。它将尝试使用 `module.__name__`,`module.__file__` 和`module.__loader__` 输入到该表示中，并默认缺失的任何信息。

以下是所使用的确切的规则：  

- 假如模块有一个 `__spec__` 属性，该分支上的信息被用作产生表示。“name”, “loader”, “origin”, 和 “has_location”这些属性将被求助。  

- 假如模块有一个 `__file__` 属性，这用作模块表示的一部分。  

- 假如模块没有 `__file__` 但确有一个不是None的 `__loader__`，那么该 加载器的表示用作模块表示的一部分。  

- 或者，仅在表示中使用模块的 `__name__`。  


*3.4版本的变化：*使用[loader.module_repr()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.module_repr)已不被推崇，而且模块分支现在被导入机制用作产生模块表示。

为了与 Python 3.3 向后兼容，在尝试以上所述任一方法前，假如定义过，将通过调用加载器的 [module_repr()](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.module_repr) 方法产生模块表示。然而，不推荐使用该方法。
##5.5. 基于路径的查找器  

如前所提及的，Python 本身带有几个默认的元路径查找器。其中之一，称作[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)([路径查找器](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder)),搜索含有一个[路径入口列表](https://docs.python.org/3/glossary.html#term-path-entry)的一个[导入路径](https://docs.python.org/3/glossary.html#term-import-path)。每个路径入口都指定一个搜索模块的位置。  

基于路径的查找器本身不知道如何导入东西。相反，它遍历单一路径入口，将每个入口与知道如何处理这种特定路径的路径入口查找器结合起来。  

路径入口查找器的默认设置实现所有在文件系统寻找模块的语义，处理类似 Python 资源代码 (`.py` 文件), Python 字节代码 (`.pyc` 和 `.pyo` 文件) 以及 共享库 (即 `.so`文件)的特殊文件型态。当标准库中 [zipimport](https://docs.python.org/3/library/zipimport.html#module-zipimport) 模块支持的话，默认路径入口查找器也处理从压缩文件中加载这些文件型态（除了共享库以外）的一切。

路径入口不需要受制于文件系统位置。他们可以引用 URLs，数据库查询，或者其他能够用一个字符串指定的位置。  

路径基础查找器提供附加的钩子程序和协议，因此你能够扩展和自定义可搜索的路径入口型态。例如，假如你想要支持路径入口作为网络 URL，你可以写一个实现 HTTP 语义并能在网络上查找模块的钩子程序。这个钩子程序（一个调用）将返回一个支持下述协议的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，其随后用作从网络获得模块的一个加载器。  

注意事项：这节和上一节都使用术语查找器，区别它们通过使用术语[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)和[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)。这两种查找器的型态非常相似，都支持相似的协议，以及在导入程序中起相似的作用，但重要的是要牢记他们稍微有所不同。尤其，元路径查找器在导入程序开始时操作，切断 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 遍历。

相比之下，在某种意义上，路径入口查找器是实现基于路径查找器的一个详述，并且事实上，如果将路径基础查找器从 sys.meta_path 中移除，没有任何路径入口查找器语义将被调用。  

###5.5.1. 路径入口查找器  

[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder) 负责查找和加载 Python 模块和包，他们可以利用一个字符串 [路径入口](https://docs.python.org/3/glossary.html#term-path-entry) 指明地址。大多数路径入口在文件系统中指定地址，但他们不必受限于此。  

作为一个元路径查找器， [基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder) 实现先前所述的 `find_spec()` 协议，然而它曝光可以用来自定义如何从导入路径查找和加载模块的附加钩子程序。  

[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)使用三个变量，[sys.path](https://docs.python.org/3/library/sys.html#sys.path)，[sys.path_hooks](https://docs.python.org/3/library/sys.html#sys.path_hooks) 和 [sys.path_importer_cache](https://docs.python.org/3/library/sys.html#sys.path_importer_cache)。 同时也使用包对象上的 `__path__` 属性。这些提供了能够自定义导入机制的其他方法。  

[sys.path](https://docs.python.org/3/library/sys.html#sys.path) 含有一个提供模块和包的搜索位置的字符串列表。它从 PYTHONPATH 环境变量和其他不同安装专用和实现专用的默认中初始化。[sys.path](https://docs.python.org/3/library/sys.html#sys.path)的入口能够指定文件系统，压缩文件和其他潜在“位置”（见[site](https://docs.python.org/3/library/site.html#module-site)模块）的目录，这些“位置”，比如 URLs, 或者数据库查询，应被用作搜索模块。[sys.path](https://docs.python.org/3/library/sys.html#sys.path) 应仅显示字符串和字节；忽略其他所有数据型态。字节入口的编码由个别[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)决定。  

[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder) 是一个[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)，所以导入机制通过调用先前所述的路径入口查找器的 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec) 方法开启导入路径 搜索。当给出 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec) 的 `path` 参数，它将是一个要遍历的字符串路径的列表，通常是包内导入中一个包的 `__path__` 属性。假如 path 变量是 `None`，这表明使用一个高阶导入和 [sys.path](https://docs.python.org/3/library/sys.html#sys.path)。  

基于路径的查找器在搜索路径中迭代每个入口，并且每个为路径入口寻找一个适当的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)。因为这可能是一个耗时的操作程序（例如，本次搜索有可能存在的 stat() 调用的耗时），基于路径的查找器维持一个路径入口查找器的缓存映射路径入口。这个缓存维持在 [sys.path_importer_cache](https://docs.python.org/3/library/sys.html#sys.path_importer_cache)（尽管名称如此，这个缓存实际上储存查找器对象而不限制于导入器对象）。因此，这个耗时搜索，即查找个别路径入口位置的路径入口查找器，仅需要完成一次即可。用户代码可以自由将缓存入口从迫使path based finder再一次执行路径入口搜索的sys.path_importer_cache中移除[[3](https://docs.python.org/3/reference/import.html#fnpic)]。  

假如路径入口不在缓存中显示，基于路径的查找器将迭代 [sys.path_hooks](https://docs.python.org/3/library/sys.html#sys.path_hooks) 中每个调用。使用单参数调用该列表中的每个[路径入口钩子程序](https://docs.python.org/3/glossary.html#term-path-entry-hook)，即要搜索的路径入口。这个调用或者可能返回一个能够处理路径入口的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder) ，或者可能引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError)异常。，基于路径的查找器使用 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常来暗示钩子程序无法找到指定[路径入口](https://docs.python.org/3/glossary.html#term-path-entry)的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)。忽略异常情况， 导入路径迭代继续。钩子程序应该预料到或者会出现一个字符串或者会出现一个字节对象；字节对象的编程取决于钩子程序（例如，它可能是一个文件系统编程，UTF-8 或其他），而且假如钩子程序不能解码参数，它会引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常。  

假如 [sys.path_hooks](https://docs.python.org/3/library/sys.html#sys.path_hooks) 迭代结束后没有[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)返回，那么基于路径的查找器的 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec) 方法将在 [sys.path_importer_cache](https://docs.python.org/3/library/sys.html#sys.path_importer_cache) 储存 `None`（表示没有该路径入口的查找器）并返回 `None`，表示该[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)不能找到模块。  

假如 [sys.path_hooks]() 的一个[路径入口钩子程序](https://docs.python.org/3/glossary.html#term-path-entry-hook)调用返回一个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，则使用下面的协议用作向查找器请求一个加载模块时使用的模块分支。  
###5.5.2. 路径入口查找器协议  
为了支持导入模块和初始化包以及扩充命名空间，路径入口查找器必须实现 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec) 方法。  

[find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec) 方法需要两个参数，导入模块的全限定名，以及（可选）目标模块。`find_spec()` 返回完全填充的模块分支。该分支将一直拥有“加载器”设置（但有一个例外）。  


导入机制中，分支代表命名空间的一[部分](https://docs.python.org/3/glossary.html#term-portion)。路径入口查找器设置分支的 “loader” 为 `None` 并且设置 “submodule_search_locations” 为一个包含该部分的列表。  

*3.4版本的变化：* [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec) 替代了 [find_loader()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_loader) 和 [find_module()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_module)，这两个现在都不被推荐使用，但在没有定义 `find_spec()` 时扔将使用。

早期的路径入口查找器可能实现这两个不被推崇的函数之一，而代替 find_spec()。为了向后兼容，这些方法仍被赞誉。但是如果在路径入口查找器上实现 find_spec()，那么遗留的方法将被忽略。  

[find_loader()](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_loader)需要一个参数，即导入模块的全限定名。`find_loader()` 返回一个二元运算符，其中第一项是加载器，第二项是一个命名空间的部分。当第一项(即加载器)是 `None` 时，这表明即使路径入口查找器没有指定模块的加载器，但它知道路径入口扩展指定模块的命名空间[部分]。这种情况几乎一直存在于请求Python导入没有物理显示的命名空间包的文件系统时。路径入口查找器返回加载器的值为None时，二元运算符的返回值的第二项必须是一个序列，尽管它可以是空的。  

假如 `find_loader()`返回一个不是 `None` 的加载器的值，该部分忽略，并且从基于路径查找器返回加载器，并通过路径入口结束搜索。

为了向后兼容其他导入协议的实现程序，许多路径入口查找器同时支持与元路径查找器支持的相同的、传统的 `find_module()` 方法。然而路径入口查找器的 `find_module()` 方法从来没有用一个 `path` 参数调用（他们预计会记录路径钩子程序的初始调用的准确路径信息）。  

由于不允许路径入口查找器扩展命名空间包，因此路径入口查找器 上的 `find_module()` 方法不被推荐使用。如果 `find_loader()` 和 `find_module()` 同时存在于一个路径入口查找器，导入系统将一直优先调用 `find_loader()` 而不是 `find_module()`。  

##5.6. 代替标准导入系统  

最可靠的完全替代导入系统的机制是删除 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 的默认内容，用一个自定义的元路径钩子程序完全代替他们。  

如仅改变导入语句的行为而不影响访问导入系统的其他 APIs 是可以接受的话，那么代替内建 `__import__()` 函数可能就足够了。也可以在模块级上采用这个技术仅在该模块中改变导入语句行为。  

在元路径早期，钩子程序选择性地阻止一些模块的导入（而不是完全将标准导入系统无效），直接在 [find_spec()](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec) 引发 [ImportError](https://docs.python.org/3/library/exceptions.html#ImportError) 异常而不是返回 `None`。后者表明元路径搜索应该继续，而引发异常情况将立即终止程序。
##5.7. __main__   
[__main__](https://docs.python.org/3/library/__main__.html#module-__main__) 模块相对 Python 导入系统是一个特殊案例。像 [elsewhere](https://docs.python.org/3/reference/toplevel_components.html#programs) 中明确的，更像 [sys](https://docs.python.org/3/library/sys.html#module-sys) 和 [builtins](https://docs.python.org/3/library/builtins.html#module-builtins)， `__main__` 模块在解释器启动中直接初始化。然而，不像这两个，它不是严格意义上的内建模块。这是因为 `__main__` 初始化的方法取决于标志和用来调用解释器的其他选项。
###5.7.1. __main__.__spec__  

根据初始化 [__main__](https://docs.python.org/3/library/__main__.html#module-__main__) 的方式，`__main__.__spec__` 获得正确地设置或者设置为 `None`。

Python 以[-m](https://docs.python.org/3/using/cmdline.html#cmdoption-m) 选项开始时，`__spec__` 设置为相应模块或包的模块分支。当加载 `__main__` 模块作为部分执行目录，压缩文件或其他 [sys.path](https://docs.python.org/3/library/sys.html#sys.path)入口时，`__spec__` 也将密集。

在[其他情况](https://docs.python.org/3/using/cmdline.html#using-on-interface-options)下，`__main__.__spec__` 设置为 `None`，因为密集 [__main__](https://docs.python.org/3/library/__main__.html#module-__main__) 使用的的代码不直接与可导入模块相对应。  
- 交互提示  
- -c 交换  
- 标准输入  
- 直接运行资源或字节代码文件  

要注意在最后的案例中 `__main__.__spec__`一直是 `None`，即使技术上文件可以作为模块直接导入。假如在 [__main__](https://docs.python.org/3/library/__main__.html#module-__main__) 中需要有效的模块元数据，则使用 [-m](https://docs.python.org/3/using/cmdline.html#cmdoption-m) 交换。  

也要注意，即使当 `__main__` 对应一个可导入模块并且对应设置 `__main__.__spec__`，但仍认为他们是远程模块。这是因为 `if __name__ == "__main__"` 保护的区块：检查仅在模块用作密集 `__main__` 命名空间时执行，且不是在常规的导入程序中。  

##5.8. 开放式问题  

XXX 做一个图表真的很有帮助。  

XXX *（import_machinery.rst）一个区域仅做模块和包的属性，或许扩展或取代数据模块参考页的相关数据条目，怎么样？  

XXX runpy, pkgutil 等人在图书馆手册都可以获得在上方的 “See Also” 连接，其指向新的导入系统部分。  

XXX 增加有关 `__main__` 初始化不同方式的更多解释？  

XXX 增加有关 `__main__` 异常/陷阱的更多信息（即从 [PEP 395](http://www.python.org/dev/peps/pep-0395) 复制）？  

##5.9. 参考  

自 Python 早期开始至今，导入机制已进化地相当大了。虽然自从写文档起一些细节已经改变了，但最初的[包说明](http://legacy.python.org/doc/essays/packages.html)仍可读。  

 [sys.meta_path](https://docs.python.org/3/library/sys.html#sys.meta_path) 最初的说明在 [PEP 302](http://www.python.org/dev/peps/pep-0302)，[PEP 420](http://www.python.org/dev/peps/pep-0420) 紧接着有展开。  

[PEP 420](http://www.python.org/dev/peps/pep-0420) 介绍了 Python 3.3 的[命名空间包](https://docs.python.org/3/glossary.html#term-namespace-package)。[PEP 420](http://www.python.org/dev/peps/pep-0420) 也介绍了替代 `find_module()`的 `find_loader()`。  

[PEP 366](http://www.python.org/dev/peps/pep-0366) 描述了主模块中明确相关导入的 `__package__` 属性的附加。  

[PEP 328](http://www.python.org/dev/peps/pep-0328) 介绍了绝对且明确的相关导入，以及 [PEP 366](http://www.python.org/dev/peps/pep-0366) 将最终说明包的语义的初始建议的 `__name__`。  

[PEP 338](http://www.python.org/dev/peps/pep-0338) 定义执行模块为脚本。  

[PEP 451](http://www.python.org/dev/peps/pep-0451) 在分支对象增加每个模块导入语句的封装。它也卸载加载机的大部分程式化任务到导入机制上。这些改变允许导入系统几个 API 的淘汰，同时也允许增加查找器和加载器的新方法。  



**脚注**  
[1]参考 [types.ModuleType](https://docs.python.org/3/library/types.html#types.ModuleType)。  

[2]实现导入库避免直接使用返回数值。反而，它通过查找 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中的模块名称获得模块对象。这间接影响的是导入的模块可能替代 [sys.modules](https://docs.python.org/3/library/sys.html#sys.modules) 中的它本身。这就是在其他 Python 实现中没有保证运行的特定的实现行为。  

[3]在遗留的代码中，可能发现 [sys.path_importer_cache](https://docs.python.org/3/library/sys.html#sys.path_importer_cache) 中 [imp.NullImporter](https://docs.python.org/3/library/imp.html#imp.NullImporter) 的实例。推荐改变代码而使用 `None` 代替。更多细节请见 [Porting Python code](https://docs.python.org/3/whatsnew/3.3.html#portingpythoncode)。   


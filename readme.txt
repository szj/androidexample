说在前面安能饭否的代码不算复杂，架构也不算特别优秀。我不打算通过画类图或者架构图的形式来说明。而是尝试用文字来说清楚代码的总体结构、相互关系及演化历史。至于细节，可以直接查看代码，或者提出疑问。当然有需要改进之处也可以提出来讨论，定出更好的方案的话就着手进行改进。 
另外，因为项目中的代码并非我一个人所为，所以难免有些不是我自己参与的部分我会解释不清或者解释错误，也烦请参与者或者有自己见解的也提出自己的见解。 
基础项目的选择安能饭否并不是一个从头编写的项目。这是我第N次强调这一点了。这一点会导致我们的代码有若干迁就和不合理之处，这是遗留代码所带来的。 
当初打算开始做安能饭否的时候，我的第一想法就是找现成的twitter客户端来改。（因为之前我也没有做过完整的android项目，从头开始有点难度。）于是在google code上找了一下，最初是找到了 andtweet (http://code.google.com/p/andtweet/) 这个项目。这个项目其实做的是很完善的，代码架构也算是不错。但是对我来说它太复杂了，改了一个多星期也没改出一个能用的玩意，反而越改越混乱。最终我不得不放弃了。 
然后又找了很多其他的项目，但是要么功能太过简单要么代码不完整，总之都不如意。最后找到了现在的这个基础项目 twitta (http://code.google.com/p/twitta/) 。当初选择这个项目的原因有两个：第一，它提供了基本完整的功能；第二，它很容易修改。我只花了一个晚上的时间就改出了基本能用的版本。再加上后面的完善和增加功能，大概只用了一个星期就弄出了一个自己可用的东西了。 
但是这个项目也不是没有缺点，实际上它有很多缺点。它的代码结构并不好，最初的时候所有代码都堆在一起，而且整体架构也不算很好。不过在开始的时候，我并没有打算把这个程序做成什么样，只是想弄个自己能用的东西出来而已，所以根本就没有重构代码的打算，十分迁就的把功能给赶出来了。 
一个典型的例子就是当时的twitta只有一个TwitterActivity，同时用于TwitterList和MentionList——因为在Twitter里，UserID和UserName是一致的，从消息本身就可以获知是否mention——不过这个在饭否行不通，于是我就copy了一份TwitterActivity，改了个名字叫MentionActivity，把里面的原来是twitter的内容都改成mention，就这么搞出了一个mention功能。 
并且当时我对android还有很多不了解的地方，也没有时间去系统的学习，就是在胡改瞎改中慢慢摸索慢慢领悟的。 
所以第一个版本，代码是非常糟糕的。 
后来三日坊主给了我一份新的界面。我看了之后相当激动，二话不说就把他拉到了项目里来。然后我开始有了想好好的做一下这个项目的想法。于是开始较为认真的对待代码了。（之所以是“较为”，是因为毕竟这是一个业余项目，时间真的有限） 
几次演化之后，才形成了今天的代码结构。接下来我会介绍一下目前的代码结构，并顺便介绍一下一些必要的演化史、设计思路、注意点及可改进之处。 
代码结构介绍包结构com.ch_linghu.fanfoudroid: 总包名，存放Application和各个实际的功能Activity 
com.ch_linghu.fanfoudroid.data: 交互用的数据封装 
com.ch_linghu.fanfoudroid.data.db: 数据库相关 
com.ch_linghu.fanfoudroid.ui.base: Activity的基类 
com.ch_linghu.fanfoudroid.ui.module: 非Activity的UI组件（Menu、ListAdapter，等等） 
com.ch_linghu.fanfoudroid.service: 后台提醒等服务相关 
com.ch_linghu.fanfoudroid.task: 并发任务（AsyncTask封装）相关 
com.ch_linghu.fanfoudroid.helper: 各种辅助函数、类 
com.ch_linghu.fanfoudroid.weibo: API封装 
com.ch_linghu.fanfoudroid.http: API封装所需的网络操作相关的封装 
其他非com.ch_linghu.fanfoudroid开头的包属于第三方包。 
Activity结构目前整个系统，几乎全部的Activity都派生自BaseActivity（有几个特殊的下面说）。BaseActivity包含了统一的对登录状态的判断及跳转、统一的OptionMenu。三日坊主引入新的界面之后，增加了一个WithHeaderActivity，这个Activity定义了Header的各种形式以及Header上各个按钮的操作。（顺便说一句，我认为这个WithHeaderActivity可以考虑再做一层抽象，以应付越来越多的Header种类。） 
从WithHeader往下，所有跟消息相关，都抽象成了TwitterListBaseActivity和TwitterCursorBaseActivity。而跟UserList相关的（目前主要就是Follower和Following），就被以类似的手法抽象成了UserListBaseActivity和UserCursorBaseActivity/UserArrayBaseActivity（其实按照TweetList的设计，是没有必要单独存在ArrayBase的，直接从ListBase派生就好），UserList的相关工作都是dodo la完成的。dodo la可以补充一下UserList部分的资料。 
跟上述两类无关的页面，除了下面提到的几个特殊的，其他都是直接派生自WithHeader。比如DM、Write、WriteDM等。 
有几个特殊的页面，是BaseActivity体系之外的： 
LoginActivity: 直接派生自Activity，用于处理登录事项。 
PreferencesActivity: 派生自PreferenceActivity，用于处理配置选项。（顺便说一句，这两个名字我觉得太过接近了，是不是把我们的类名字改一下） 
AboutDialog: 这实际上并不是一个Activity，它只是一个helper类，提供了一个show函数，用于显示一个基本的关于对话框。我曾经在issue里写过一条，AboutDialog谁有空可以重新设计一下。 
消息列表抽象类设计【注】目前对消息的称呼，在项目代码中有两种不同的名字：Status和Tweet，可以考虑统一起来。 
在整个系统中，消息列表可以说是最常见的。每个消息列表来源各不相同，有搜索的、有随便看看的，有提到自己的、有用户的，它们有些直接来自API，有些来自API和本地缓存。但是它们都有一些共同的操作，比如都可以回复、转发、收藏，等等。因此，有必要为所有的消息列表页面做一个抽象，并且抽象要能满足各种需要，但是要让新页面的编写变得比较方便。 
基于以上考虑，我们对消息列表页面做了两层抽象。 
我们做了两个层次的封装，第一层是TwitterListBaseActivity，这一层是仅仅假设页面上有一个Tweets List，那么我们可以确定需要显示的消息内容（头像、ScreenName、消息文本、时间、来源等），但是对于数据来源我们不做任何假设。这个基类可以用于派生各种跟Twitter消息列表有关的页面。包括那些消息是临时产生的，不经过数据库的页面，比如搜索结果、User timeline之类。 
我们为这个基类定义了一系列抽象函数，交给派生类去实现。而基类本身，利用这些抽象函数进行框架性的操作。 
/**  * 用于指定页面对应的layout  */ abstract protected int getLayoutId();  /**  * 用于指定页面上的ListView，这个抽象主要是考虑到有一些页面可能会使用   * ListView的扩展类。比如SearchResultActivity，用的就是  * MyListView(可以自动获取更多结果)  */ abstract protected ListView getTweetList();  /**  * 用于指定ListView对应的Adapter  */ abstract protected TweetAdapter getTweetAdapter();  /**  * 完成初始化设置  * FIXME:这个函数应该是从原始代码中遗留下来的，主要目的是在对TweetList操作  * 之前完成一些初始化的工作。  * 但是我在实际使用的时候感觉这个函数的实现其实不太好写，因为时间点控制太  * 严格。考虑是不是可以去掉这个函数，仅仅使用 _onCreate ？  */ abstract protected void setupState();  /**  * 指定页面标题。  * 目前这个函数没有被用到。可以考虑去掉，或者把它用起来。  */ abstract protected String getActivityTitle();  /**  * 指定是否使用基本菜单。  * 我们在TwitterListBaseActivity中定义了一套针对每一条消息的基本菜单  * 主要有：xxx的空间、回复、热饭、私信、收藏/取消收藏 操作  * 这样可以方便各个派生类的弹出菜单基本保持一致。（派生类可以在这几项  * 后面增加更多菜单项，无须把useBasicMenu设置成false）  * 但是考虑到某些页面可能不需要这些菜单，或者需要使用完全不同的菜单  * 我们做了这样一个函数，如果不需要的话可以让这个函数返回false，这样就不会  * 有任何默认菜单，你可以从头构建自己的菜单项  */ abstract protected boolean useBasicMenu();  /**  * 用于返回指定位置的Tweet数据  * Position是Item选择或弹出菜单选择时传入的，  * item在整个List中的绝对位置  */ abstract protected Tweet getContextItemTweet(int position);  /**  * 没有实际用到。我也忘记放在这里干吗用的了。  * 考虑删掉  */ abstract protected void updateTweet(Tweet tweet);考虑到有不少页面是直接从数据库获得数据，它们的操作几乎是一致的，仅仅是数据库的查询条件不同。我们在TwitterListBaseActivity的基础上又派生了一个新的抽象类：TwitterCursorBaseActivity，专门用于跟数据库关联的tweet list页面。它从TwitterListBaseActivity派生，对上面提到的那些抽象函数做了自己的实现。然后又给出了自己的抽象函数： 
/**  * 标记全部记录为“已读”  * “已读”标记是twitta中引入的概念，但是实际上包括twitta自己，  * 以及后来我们的代码，都没有用到这个标记。  * 因此，这里的操作纯粹是浪费时间，可以考虑暂时忽略。  */ abstract protected void markAllRead();  /**  * 获得“全部”消息。这个函数实际上就定义了查询条件  * 查询条件的不同影响到列表数据的不同，  * 这个函数的定义是不同页面最重要的区别  */ abstract protected Cursor fetchMessages();  /**  * 获得数据类型。  * 目前我们把全部的消息数据都存放在一张表(StatusTable)中，  * 通过StatusType来区分。这个函数就要求指定所操作的StatusType  * 这里有两点，第一是名字应该考虑改成getStatusType  * 第二是要考虑有没有混合type操作的可能。（目前为止还没有）  */ public abstract int getDatabaseType();  /**  * 获得页面操作的数据的所有者。大部分页面的所有者都是自己(myself)  * 但是因为引入了user profile页面，因此有些页面，例如user timeline、  * fav timeline等，它的所有者可能是其他user，这时就需要用这个函数来指定，  * 以方便操作。BTW，这个名字似乎也应该修改成更加明确的名字。  * (目前user timeline没有从CursorBase继承，因此只有fav timeline实际用到了)  */ public abstract String getUserId();  /**  * 获得最新一条消息的ID  */ public abstract String fetchMaxId();  /**  * 获得最早一条消息的ID  */ public abstract String fetchMinId();  /**  * 把Tweet列表信息存入数据库  * Tweet列表信息通常是通过API获得的  */ public abstract int addMessages(ArrayList<Tweet> tweets,                 boolean isUnread);  /**  * 获得比指定消息更新的消息  * 主要用于“刷新”操作  */ public abstract List<Status> getMessageSinceId(String maxId)   /**  * 获得比指定消息更早的消息  * 主要用于“更多”操作  */ public abstract List<Status> getMoreMessageFromId(String minId)API的封装twitta的API封装是非常简陋的。三日坊主给项目带来了全新的、功能更加完善的API。API在接下来的分离包的工作中被归到了com.ch_linghu.fanfoudroid.weibo和com.ch_linghu.fanfoudroid.http两个包中。API的封装工作完全是由三日坊主完成的，使用的实例也可以在代码中随处可见。如果有什么问题可以直接跟三日坊主交流。也希望三日坊主能在这里补充一下相关的资料及注意点之类。尤其是异常部分的设计，相信接下来会非常用得到。 
API和HTTPcom.cn_linghu.fanfoudroid.weibo 的所有API方法都是在使用 com.cn_linghu.fanfoudroid.http 的 httpClient 进行 POST 或 GET 请求. 
理想状态是使用者只需要使用 weibo 类的接口即可, 而无需关心 httpClient 的具体实现. 
API异常所有 httpClient 进行的请求都有可能抛出 HttpException , 其中分两种: 
底层异常: 由底层函数抛出的异常, 可以通过 getCause() 查看具体为哪种异常, 但一般情况并不需要关注此类注底层异常, 可统一视其为请求内部异常. 具体原因: 
URISyntaxException, 由new URI 引发. 
IOException, 由createMultipartEntity 或 UrlEncodedFormEntity 引起. 
IOException和ClientProtocolException, 由HttpClient.execute 引发. 
# 
子类异常: HTTP CODE非200所导致的异常, 此类异常都是 HttpException 子类, 可统一捕捉. 
HttpRequestException, 通常发生在请求的错误,如请求错误了 网址导致404等, 抛出此异常, 首先检查request log, 确认不是人为错误导致请求失败. 
HttpAuthException, 通常发生在Auth失败, 检查用于验证登录的用户名/密码/KEY等. 
HttpRefusedException, 通常发生在服务器接受到请求, 但拒绝请求, 可是多种原因, 具体原因服务器会返回拒绝理由, 调用HttpRefusedException#getError#getMessage查看. 
HttpServerException, 通常发生在服务器发生错误时, 检查服务器端是否在正常提供服务. 
HttpException, 其他未知错误. 

捕捉说明
一般情况下, 使用API方法的时候只会特别关心抛出的少数的几种异常, 比如是否登录成功的 HttpAuthException , 是否有权限查看某个用户信息的 HttpRefusedException . 而对于其他的所有异常都并不关系, 不需要传递给用户特别的消息, 只需要告之 "请求失败" 之类的即可, 然后将错误写入日志, 由开发人员进行分析. 
因此, 默认的异常捕捉基本都是: 
  try {       status = getApi().showStatus(reply_id);   } catch (HttpException e) {     Log.e(TAG, e.getMessage(), e);     return TaskResult.IO_ERROR;   }如果需要单独关注某个异常, 则提前捕捉你所关心的异常: 
  try {     status = getApi().showStatus(reply_id);   } catch (HttpAuthException auth_e) {     // 用户身份验证失败    } catch (HttpRefusedException auth_e) {     // 可能没有权限查看该条信息    } catch (HttpException e) {     // 其他情况引起的异常      Log.e(TAG, e.getMessage(), e);     return TaskResult.IO_ERROR;   }数据封装系统里对于数据的封装，分为几层：通过API获得的json数据，在API封装层中被直接包装成Entity结构。这些Entity结构的实现都在com.ch_linghu.fanfoudroid.weibo中，主要包括以下类： 
Status: 对消息的封装 
DirectMessage: 对私信的封装 
User: 对用户信息的封装 
Photo: 对消息中照片信息的封装 
Trend: 对单个热门话题的封装 
SavedSearch: 对保存搜索结果的封装 
Trends: 对热门话题的封装（Trend的列表） 
IDs: 用户列表的封装 

以上这些数据类封装是直接针对json数据的，是只读的。实际使用中经常会受到限制。因此我们定义了一套纯粹的、与数据来源无关的data结构。这些结构存放在com.ch_linghu.fanfoudroid.data中。主要包括： 
Tweet: 对消息的封装 
Dm: 对私信的封装 
User: 对用户信息的封装 

我们从API获得数据之后，立刻将它们转换为data结构，之后在程序中主要是以data结构做操作。有部分较简单的数据，如Trends、SavedSearch等，是直接用Entity结构进行处理的。 
我们可以看到，在程序中对数据的处理来自两个不同的层次：entity层和data层，并且两个层次的逻辑意义并没有明显区别，完全是实现上的限制。因此，我认为，在以后的改进中，可以考虑增强entity层，使之可以完全承担data层的工作，然后消除data层，这样可以避免不必要的转换工作，并使得数据处理的逻辑更加清晰。 
为了缓存和持久化，我们还定义了数据库层，数据库定义都在com.ch_linghu.fanfoudroid.data.db中，目前我们主要定义了以下表: 
StatusTable: 消息表。所有的消息都存放在这个表中，除了Status本身的字段外，我们还定义了一些其他的字段以方便操作： 
status_type: 用于标示消息的类型。注意消息类型是不重叠的，因此同一个ID的消息，在表中可能存在多条，分别具有不同的status_type。 
is_unread: 用于标示消息的已读/未读状态。目前该字段并没有被实际使用。 
owner: 用于标示消息的所有者，以便处理不同用户的同一个status_type的消息，如favourite timeline。 
MessageTable: 私信表。用于存放私信。 
UserInfoTable: 用户信息表。用于存放用户信息。 
StatusDatabase: 提供了DatabaseHelper及Table的操作接口。 

并且我们提供了数据库和data结构相互操作的接口。这些接口都定义在StatusDatabase中。 
现在各个表本身的定义已经分离到各个单独的类中。但是所有表的操作接口仍然都放在StatusDatabase里，这是不合理的做法，未来应该要把各个接口也分别放到各自的类中。 
目前全部的数据表我们都认为是缓存表，里面的数据都不是关键信息，都是可以消失的。目前Status表的策略是同一个owner、同一个status_type的消息，在无法保证数据连续的情况下，仅保留最新20条。其他表目前没有采用这一策略，但是也可以这样做。 
目前系统仍然缺乏一个统一的全局数据的封装，例如登录用户的相关扩展信息、全局调试开关、全局配置信息等。目前这些信息分别存放在Preference、TwitterApplication、weibo.Configuration等处，我觉得应该有一个更加统一的处理机制。可以考虑在Preference上进行再次扩展。 
TaskAsyncTask是Android提供的异步执行机制，可以方便的将一些工作转移到新的线程里去执行，执行过程中或完毕后调用回调函数进行界面更新及其他处理工作。（因为UI的操作是不能在线程中做的，这不仅仅是Android的限制，也是几乎所有GUI系统的限制） 
在本项目中，需要异步执行的地方很多，基本的Task模式是这样： 
在doXXX函数中首先判断是否当前任务是否正在运行。如果是则直接退出，否则新建一个任务并用execute方法启动之。 顺便说一句，现有的逻辑会造成Task被重复new，这里需要修改。 
Task启动前先执行onPreExecute，然后后台执行doBackground，并在执行过程中通过publishProgress向主线程报告进度，主线程在onProgressUpdate回调中进行处理。 
执行结束视情况调用onPostExecute或onCancelled。 
在Activity被Destory时，我们要cancel所有正在运行的Task，以防止Task的重入。 
为了统一起见，我们对AsyncTask做了一点薄封装(GenericTask)。我们主要做了以下两点工作： 
简化了重载函数。AsyncTask需要重载4个函数，GenericTask只需要重载一个（doBackground）。 
将主线程回调函数分离到TaskListener中以便复用。一个Activity中可能需要实现好几个Task，但是他们调用结束后可能是做相同的界面刷新动作，这样我们只需要一个TaskListener，挂到不同的Task去，就可以了，无需每个Task写重复的代码。关于这一点目前做得并不算好，几乎每一个Task仍然都有自己独立的Listener，虽然这样也可以，但是我觉得一定有可以精简合并的，可以考虑精简一下。 
现有的封装并没有能解决doXXX函数中的重复代码（判断当前任务是否正在运行），及Destory时对Task的退出处理。这里是需要改进的。 
【注】实际上Destory时对Task的退出处理是有考虑到的。三日坊主曾经写过一个TaskManager来尝试对Task的cancel进行统一处理。不过我认为那个方案还有可商榷之处，因此目前还没有大规模使用。 
此外，关于Task还有几个辅助类： 
TaskParam: 用于Task的信息传入。 
TaskResult: 用于Task的返回值。 
TaskAdapter: 对TaskListener的进一步简化，为TaskListener的接口方法做了默认实现，这样派生自TaskAdapter的类就只重载需要的方法，不需要去实现每一个方法了。 
TweetCommonTask: 对于单条消息的一些操作实在太常用（目前有Delete和Favourite两个操作），因此我们做了一个Common实现，免得在各个Activity里重复实现。 

Service对于Service部分我没有做仔细的阅读，这里基本是直接沿用了twitta的代码。如果有谁觉得对这块比较了解可以来解释一下代码的请帮助更新这一部分。 
辅助工具类这里定义了项目中大量被使用的工具方法和类。 
图像缓存图像缓存由一系列相互关联的类构成。主要完成对头像、照片的缓存和处理工作。 
首先我们看ImageCache，这是一个图像缓存的接口，主要用于将URL和Bitmap对应起来。它提供了两个方法：get和put。put将一个url和一个bitmap关联起来，get则是获取指定url对应的bitmap。 
ImageCache有两个派生类：MemoryImageCache和ImageManager。其中MemoryImageCache较为简单，它直接使用了HashMap在内存中保存url和bitmap的关联关系。这里就不多做解释了。 
ImageManager本身也是一个ImageCache，它也实现了put和get接口，也可以用作图像缓存，只不过它的图像是保存成文件的（目前没有保存到SD卡，而是保存在手机的应用程序私有目录中）。但是它做的工作比这个要多很多。 
ImageCache的put带有两个参数：url和bitmap。而ImageManager额外实现了更多的put，有单个url参数的，这个函数可以自动从网络获取指定url的图片然后保存，有指定图像质量的，这可以压缩保存图片。还有指定file、bitmap和quality的，可以用于将bitmap直接以指定quality存入指定文件。它还提供了compressImage和resizeImage两个辅助函数，用于压缩和缩放图像（在WriteActivity的照片上传部分会用到）。 
另外，为了方便头像的管理，我们还另外实现了一个ProfileImageCacheManager，这个类专门用于管理头像。它只提供了一个对外接口：get，这个方法接受两个参数：url和callback。作用是得到指定url的头像bitmap。ProfileImageCacheManager的内部使用ImageManager做图片管理。get方法首先会去ImageManager尝试获得对应的图片。如果这个图片存在，则直接返回对应的Bitmap，callback被忽略。如果图片不存在，则会启动一个Task，从网络获取指定URL的图片，并存入ImageManager中。当Task执行完毕，图片被正确获得之后，将调用callback的refresh函数刷新界面。至于如何刷新，由callback自己实现。 
UtilsUtils中存放了各种辅助函数。最初是twitta实现的，当初它的作者可能没有想那么多，因此Utils里堆了很多实际上用途各异的东西。后来因为我们项目的需要，又增加了一些，我认为这些可以在接下来做一次重构，把它们细分到各个意义更明确的类里去。这个难度应该不会太大。 
Utils中的辅助函数大体可以分成: 
字符串辅助函数： 
isEmpty 判断字符串是否为空 
日期转换类函数: 
parseDateTime - 
parseDateTimeFromSqlite |-将字符串转换成日期类 
parseSearchApiDateTime - 
getRelativeDate 得到类似“大约xx分钟前”这样的字符串 
getNowTime 获得当前时间 
消息文本处理函数 
setSimpleTweetText 设置控件显示纯文本的消息 
setTweetText 设置控件显示带格式和跳转链接的消息 
linkfyxxx函数簇 用于自定义linkfy过滤器，产生需要的链接内容。如搜索、用户名、URL等 
图像处理函数 
drawableToBitmap 将drawable资源的内容转换成Bitmap 
照片预览相关处理函数 
getPhotoPageLink 获得消息中的照片链接 
getPhotoURL 获得照片页面中的图片地址 

总体运作流程
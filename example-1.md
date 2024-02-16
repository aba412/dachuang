# **_如何使用 ICASAR 软件处理合成数据_**
1. 生成随机的合成干涉图日期
1. 导入合成数据
1. 制作合成时间序列并进行可视化展示。
1. 进行独立成分分析（ICA）并输出分解结果


### *一、生成随机的合成干涉图日期*
```
def generate_random_ifg_dates(n_dates):   #定义一个生成随机干涉图日期的函数，形参n_dates是一个用来指定生成随机日期数量的整数变量
‘’‘
以下定义daisy_chain_from_acquisitions函数，其接受一个列表作为输入，然后根据列表中的日期生成一系列干涉图的名称。
形参acquistions数据类型为list，为一系列YYYYMMDD格式的卫星拍摄或采集相应数据的日期，acquisitions列表存储了一系列获取日期，通过这些获取日期生成了干涉图的名称
返回值daisy_chain为list，其中包含了生成的干涉图的名称
’‘’
    def daisy_chain_from_acquisitions(acquistiions):
        daisy_chain = []
        n_acqs = len(acquisitions)  #n_acqs为整型变量，表示输入的acquisitions列表中日期的数量帮助程序针对不同长度的日期列表生成正确数量的干涉图名称
        for i in range(n_acqs-1):
            daisy_chain.append(f"{acquisitions[i]}_{acquisitions[i+1]}") #将生成的干涉图名称添加到 daisy_chain 列表中
        return daisy_chain
    
    import datetime as dt
    import random
    day0 = "20150101"  #初始日期
    day0_dt = dt.datetime.strptime(day0, '%Y%m%d')  #将day0转为datetime形式
    acq_dates_dt = [day0_dt]  #一个存储生成的随机日期的列表，其中每个日期都以datetime对象的形式表示
    for i in range(n_dates):
        t_baseline = random.choice([6,12,18,24])  #一个用于存储每次循环迭代中随机选择的时间基线的变量。使用这个变量来计算随机生成的日期。
        acq_dates_dt.append(acq_dates_dt[-1] + dt.timedelta(days = t_baseline)) #计算出下一幅 SAR 影像的获取日期，并将其添加到日期时间对象列表中
    acq_dates = [dt.datetime.strftime(i, '%Y%m%d') for i in acq_dates_dt] #将日期时间对象列表 acq_dates_dt 中的每个日期时间对象转换为指定格式的日期字符串，并存储在 acq_dates 列表中
    
    ifg_dates = daisy_chain_from_acquisitions(acq_dates)
    return ifg_dates  #返回生成的干涉图像名称序列
```

### *二、导入合成数据*
```
# 定义一些参数，包括要恢复的主成分数量、bootstrapping 参数、tsne 参数等等。
ICASAR_settings = {"n_comp" : 5,                                        
                    "bootstrapping_param" : (200, 0),                   
                    "hdbscan_param" : (35, 10),                        
                    "tsne_param" : (30, 12),                            
                    "ica_param" : (1e-2, 150),                          
                    "hdbscan_param" : (35,10),                          
                    "out_folder" : Path('example_spatial_01_outputs'),  
                    "load_fastICA_results" : True,
                    "figures" : "png+window"}
# 导入数据，包括时间序列、合成空间源、噪声、像素掩膜、经度、纬度等。
with open('synthetic_data.pkl', 'rb') as f:  #以二进制只读模式打开synthetic_data.pkl文件并赋值给f
#以下操作为将f中存储的数据反序列化为原始对象，并加载到相应的变量中
    A_dc = pickle.load(f)                                           
    S_synth = pickle.load(f)                                        
    N_dc = pickle.load(f)                                           
    pixel_mask = pickle.load(f)                                     
    lons = pickle.load(f)                                           
    lats = pickle.load(f)            
```

### *三、制作合成时间序列并进行可视化展示*
```
X_dc = A_dc @ S_synth + N_dc    #计算包含合成源信号以及噪声的混合数据。A_dc 是混合矩阵，S_synth 是源信号矩阵，N_dc 是噪声矩阵。@ 符号表示矩阵乘法。                                             
phUnw = X_dc                                                                    
fig1, axes = plt.subplots(2,3)                                                  
for i in range(3):
    axes[0,i].imshow(col_to_ma(S_synth[i,:], pixel_mask))      #在第一行的第 i 列中显示第 i 列的源信号
    axes[1,i].plot(range(A_dc.shape[0]), A_dc[:,i])            #在第二行的第 i 列中绘制第 i 列的混合矩阵
    axes[1,i].axhline(0)                                       #在第二行的第 i 列中添加一条水平线表示零线
fig1.suptitle('Synthetic sources and time courses')            #设置整个图形的标题为 'Synthetic sources and time courses'
fig1.canvas.manager.set_window_title("Synthetic sources and time courses")    #设置窗口标题为 "Synthetic sources and time courses"

fig2, axes = plt.subplots(2,5)              #创建一个包含 2 行 5 列子图的 Figure 对象 fig2，并在每个子图中显示混合后的数据。                                             
for i, ax in enumerate(np.ravel(axes[:])):
    ax.imshow(col_to_ma(phUnw[i,:], pixel_mask))
fig2.suptitle('Mixtures (intererograms)')                         #设置整个图形的标题为 'Mixtures (intererograms)'
fig2.canvas.manager.set_window_title("Mixtures (intererograms)")  #设置窗口标题为 "Mixtures (intererograms)"

fig3, axes = plt.subplots(1,3, figsize = (11,4))                  #创建一个包含 1 行 3 列子图，设置其尺寸为 (11, 4) 的 Figure 对象 fig3，并在每个子图中显示不同的数据              
axes[0].imshow(X_dc, aspect = 500)                                #显示混合数据矩阵，设置横纵比为 500
axes[0].set_title('Data matix')                                   #设置第一个子图的标题为 'Data matrix'
axes[1].imshow(pixel_mask)                                        #显示像素掩模
axes[1].set_title('Mask')                                         #设置第二个子图的标题为 'Mask'
axes[2].imshow(col_to_ma(X_dc[0,:], pixel_mask))                  #显示第一行的干涉图
axes[2].set_title('Interferogram 1')                              #设置第三个子图的标题为 'Interferogram 1'
fig3.canvas.manager.set_window_title("Interferograms as row vectors and a mask")   #设置窗口标题为 "Interferograms as row vectors and a mask"

```

### *四、进行独立成分分析（ICA）并输出分解结果*
```
S_best, time_courses, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean  = ICASAR(spatial_data = spatial_data, **ICASAR_settings)
```

### *Others*
```
#创建一个名为 spatial_data 的字典，用于存储脉冲星合成孔径雷达干涉测量（InSAR）的空间数据。这些数据包括相位数据、掩码数据、经纬度坐标数据和测量日期数据
#作用是创建并组织脉冲星合成孔径雷达干涉测量的空间数据，以方便后续处理和分析
ifg_dates_dc = generate_random_ifg_dates(phUnw.shape[0])
spatial_data = {'ifgs_dc' : phUnw,
                'mask'        : pixel_mask,
                'lons'        : lons,                                           
                'lats'        : lats,                                           
                'ifg_dates_dc'   : ifg_dates_dc}
```
*字典*：在 Python 中，字典（dictionary）是一种可变容器模型，用来存储键值对（key-value pairs）。字典中的每个元素都是一个键值对，其中键（key）必须是唯一的，而值（value）可以是任意类型的对象。字典使用花括号 {} 来表示，其中每个键值对之间使用逗号 , 隔开。

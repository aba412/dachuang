# *概述*

example_spatial_02的代码主要展示了如何使用 ICASAR 处理 LiCSBAS 输出的数据.包括载入 LiCSBAS 的 .h5 文件，以及使用 ICASAR 进行独立成分分析（ICA）来提取地表变形信号。  
具体包括了对增量干涉图和累积干涉图进行不同的处理方式，以及展示了如何可视化拟合干涉图的结果。  

### 加载和准备 LiCSBAS 输出的数据，并将所需传入字典spatial_data中

```
LiCSBAS_out_folder_campi_flegrei = Path('./campi_flegrei_LiCSBAS_example_data/')  #创建一个文件夹路径对象
print(f"Opening the LiCSBAS .h5 file...", end = '')
 
displacement_r2, tbaseline_info, ref_xy = LiCSBAS_to_ICASAR(LiCSBAS_out_folder_campi_flegrei, figures=True)
#上面那行代码是在调用 LiCSBAS_to_ICASAR 函数来处理 LiCSBAS 输出的数据，
#并将返回值分别赋值给displacement_r2, tbaseline_info, ref_xy       
print(f"Done.  ")

spatial_data = {'ifgs_dc'        : displacement_r2['incremental'], #增量变形数据
                'mask'           : displacement_r2['mask'],        #遮罩数据，即掩膜
                'ifg_dates_dc'   : tbaseline_info['ifg_dates'],    #干涉图日期信息          
                'dem'            : displacement_r2['dem'],         #数字高程模型数据DEM    
                'lons'           : displacement_r2['lons'],        #经度数据
                'lats'           : displacement_r2['lats']}        #纬度数据
```

##### **示例一**
```
#%% Example 1: sICA after creating all interferograms  

ICASAR_settings = {"n_comp" : 5,                              
                    "bootstrapping_param" : (200, 0),                               
                    "tsne_param" : (30, 12),                   
                    "ica_param" : (1e-2, 150),       
                    "hdbscan_param" : (100,10),           
                    "out_folder" : Path('example_spatial_02_outputs_sICA'),   
                    "load_fastICA_results" : True,                      
                    "ifgs_format" : 'all',                   
                    'sica_tica'         : 'sica',              
                    'max_n_all_ifgs' :  1000,                 
                    'label_sources'         : True,                 
                    "figures" : "png+window"}           
                
S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels  = ICASAR(spatial_data = spatial_data, **ICASAR_settings)   #使用ICASAR 算法将对空间数据进行处理和分析，最终返回上述各个变量，提供对数据进行反演和分析的结果。

visualise_ICASAR_inversion(spatial_data['ifgs_dc'], S_ica, A_ica, displacement_r2['mask'], n_data = 10)  #对 ICASAR 反演结果进行可视化处理。
```
  该示例使用所有干涉图像进行sICA。设置了ICASAR的参数，调用ICASAR函数并返回处理后的结果。最后调用visualise_ICASAR_inversion函数，将干涉图像、S_ica、A_ica和掩膜传入，以可视化展示如何使用学习到的组件（ICs）来拟合干涉图像。  
##### **示例二**
```
#%% Example 2: sICA with only incremental interferograms

ICASAR_settings = {"n_comp" : 5,                                         
                    "bootstrapping_param" : (200, 0),                                         
                    "tsne_param" : (30, 12),                             
                    "ica_param" : (1e-2, 150),                            
                    "hdbscan_param" : (100,10),                          
                    "out_folder" : Path('example_spatial_02_outputs_sICA_incremental'),   
                    "load_fastICA_results" : True,                      
                    "ifgs_format" : 'inc',                      
                    'max_n_all_ifgs' :  1000,
                    'sica_tica'         : 'sica',                        
                    'label_sources'         : True,                     
                    "figures" : "png+window"}                            

S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 
```
该示例使用增量干涉图像进行sICA。同样地，设置ICASAR的参数，调用ICASAR函数并返回处理后的结果。
##### **示例三**
```
#%% Example 3: tICA given the incremental interferograms, but this will calculate the cumulative interferograms.

ICASAR_settings = {"n_comp" : 5,                                         
                   "bootstrapping_param" : (200, 0),                                          
                   "tsne_param" : (30, 12),                             
                   "ica_param" : (1e-2, 150),                            
                   "hdbscan_param" : (100,10),                          
                   "out_folder" : Path('example_spatial_02_outputs_tICA_incremental'),   
                   "load_fastICA_results" : True,                      
                   'max_n_all_ifgs' :  1000,
                   'sica_tica'         : 'tica',                        
                    'label_sources'         : True,                     
                   "figures" : "png+window"}                            

S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 
```
该示例使用增量干涉图像进行tICA。同样地，设置ICASAR的参数，调用ICASAR函数并返回处理后的结果。  

  

### 相关名词及涉及的算法（不分先后）简介
* 名词
  * *增量变形数据*
    * 增量变形数据（Incremental Deformation Data）是一种用于描述对象变形信息的数据类型。它记录了一个对象在不同时间或不同状态下的几何形状之间的差异，通常以一系列的位移向量或变换矩阵的形式呈现。
  * *掩膜Mask*  
      掩膜是在图像或数据处理中常用的一种技术，用于指定特定区域或像素的有效性。它是一个与原始图像或数据具有相同大小的二进制图像或布尔数组，其中每个像素或元素的值表示其有效性。  
      在遥感图像处理中，掩膜通常用于指定感兴趣区域（Region of Interest, ROI），即我们只对该区域内的像素进行分析和处理，而忽略其他区域。通过将不感兴趣的区域标记为无效，可以排除这些区域对分析结果的影响，提高处理的准确性和效率。
  * *数字高程模型DEM*
    DEM是一种用于表示地球表面的高程信息的数字化模型。
* 键名
  * *n_comp*
    * 含义：ICASAR中要恢复的独立成分（Independent Components Analysis，ICA）的数量，即要保留的主成分数目。
    * 作用：通过设置合适的主成分数量，可以控制ICASAR提取的特征维度，从而影响后续的数据分析和处理过程。选择合适的主成分数量可以保留重要信息，同时减少不必要的噪声。
  * *bootstrapping_param*
    * 含义：使用Bootstrapping方法时的参数，包括运行次数和不运行Bootstrapping的次数。
    * 作用：通过设置Bootstrapping方法的参数，可以控制在重复采样过程中的执行次数，以及不执行Bootstrapping的次数。这有助于评估估计量的精确性和稳定性。
  * *hdbscan_param*
    * 含义：HDBSCAN聚类算法中的参数，包括最小聚类大小和最小样本数。
    * 作用：通过设置HDBSCAN算法的参数，可以控制生成的聚类数量和大小。较大的最小聚类大小可以过滤掉太小的聚类，使得结果更加稳定和可靠。
  * *tsne_param*
    * 含义：t-SNE降维算法中的参数，包括困惑度和早期夸大因子。
    * 作用：通过调整t-SNE算法的参数，可以影响在降维过程中对数据点之间的关系的保留程度。这有助于保留数据的局部结构信息，并在可视化过程中提供更好的数据展示效果。
  * *ica_param*
    * 含义：ICA算法中的参数，包括容差和最大迭代次数。
    * 作用：通过设置ICA算法的参数，可以控制算法的收敛性和稳定性，以获得更好的独立成分提取效果。
  * *out_folder*
    * 含义：输出结果保存的文件夹路径。
    * 作用：指定ICASAR算法输出结果的保存路径，便于后续结果的查看和分析。
  * *load_fastICA_results*
    * 含义：是否加载已存在的FastICA运行结果。
    * 作用：通过设置该参数，可以控制是否加载已存在的FastICA运行结果，以加快ICASAR算法的执行速度。
  * *max_n_all_ifgs*
    * 含义：所有干涉图像的最大数量限制。
    * 作用：限制所有干涉图像的数量，避免处理过多数据导致算法执行效率低下。
  * *sica_tica*
    * 含义：控制空间源或时间序列是否独立的参数。
    * 作用：通过设置该参数，可以控制ICASAR算法是处理空间源还是时间序列，以满足具体问题的需求。
  * *label_sources*
    * 含义：是否尝试标识独立成分的类型。
    * 作用：通过设置该参数，可以控制ICASAR算法是否尝试识别独立成分的类型，如形变相关APS、湍流APS等，进而对结果进行更细致的分析和解释。
  * *figures*
    * 含义：输出图形的格式和方式（如.png文件、交互式matplotlib图形等）。
    * 作用：通过设置该参数，可以指定输出图形的格式和方式，便于后续的可视化和结果展示。
* 参量名
  * *S_ica*  
    sICA 算法得到的稀疏独立成分（sparse independent components），包含了反演得到的空间源信息。
  * *A_ica*  
    sICA 算法得到的混合矩阵（mixing matrix），包含了反演得到的时间序列信。
  * *Iq*  
    Iq 是 Q 矩阵，用于计算深度学习相关性。
  * *x_train_residual_ts*  
    训练集的残差时间序列。
  * *n_clusters*  
    聚类数目。
  * *S_all_info*  
    包含了所有独立源的信息。
  * *phUnw_mean*  
    无包裹相位均值。
  * *sources_labels*
    源标签。
  * *displacement_r2['mask']*
    这是位移数据的掩模信息，用于控制哪些数据点应该被考虑或排除在可视化过程中。
  * *n_data*
    用于指定要可视化的数据数量，即要显示的数据点个数。
* 涉及的算法
  * ICA独立成分分析
    ICA 是一种盲源信号分离方法，旨在从混合信号中分离出相互统计独立的原始信号。其基于统计学原理，通过最大化信号的相互独立性来分离混合信号中的原始信号。
  * *聚类算法HDBSCAN*  
    聚类算法是一种无监督学习算法，其主要目标是将数据集中的样本分成不同的组或簇，使得同一簇内的样本彼此相似，而不同簇之间的样本差异较大。
    HDBSCAN（层次密度聚类算法）就是是一种基于密度的聚类算法。，该算法不需要预先指定簇的数量就，能够自动识别不同密度的区域，从而生成具有不同数量和大小的聚类。在处理具有复杂形状和不同密度的簇时表现出色。
  * *流形学习算法t-SNE*
    流形学习是一种非线性降维技术，用于将高维数据映射到低维空间中，同时保留原始数据的拓扑结构和流形结构。与传统的线性降维方法（如主成分分析）不同，流形学习可以处理非线性关系，并且更适用于复杂的高维数据集。  
    t-SNE（t-Distributed Stochastic Neighbor Embedding）是其中的一种，旨在将高维数据映射到低维空间以进行可视化，并保留数据点之间的局部结构。与其他流形学习算法类似，t-SNE 也专注于发现数据的非线性结构和复杂关系。
    

# *概述*

example_spatial_02的代码主要展示了如何使用 ICASAR 处理 LiCSBAS 输出的数据.包括载入 LiCSBAS 的 .h5 文件，以及使用 ICASAR 进行独立成分分析（ICA）来提取地表变形信号。具体包括了对增量干涉图和累积干涉图进行不同的处理方式，以及展示了如何可视化拟合干涉图的结果。  

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

ICASAR_settings = {"n_comp" : 5,                               #sICA 算法中需要提取的稀疏独立成分的数量
                    "bootstrapping_param" : (200, 0),          #要恢复的组件数                       
                    "tsne_param" : (30, 12),                   #流形学习算法t-SNE的perplexity和early_exaggeration参数          
                    "ica_param" : (1e-2, 150),                 #ICA算法的tolerance和max iterations参数           
                    "hdbscan_param" : (100,10),                #聚类算法HDBSCAN中最小集群大小和最小样本数          
                    "out_folder" : Path('example_spatial_02_outputs_sICA'),   #输出文件夹的路径，用于保存结果
                    "load_fastICA_results" : True,             #是否加载之前保存的 FastICA 结果         
                    "ifgs_format" : 'all',                     #干涉图像格式，这里指的是增量式干涉图像
                    'sica_tica'         : 'sica',              #sICA 和 tICA（一种非线性降维方法）的选择，默认为 sICA          
                    'max_n_all_ifgs' :  1000,                  #全部干涉图像的最大数量
                    'label_sources'         : True,            #是否为源进行标签化         
                    "figures" : "png+window"}                  #输出结果的图像格式，这里是指定生成 PNG 图像和显示窗口
                
S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels  = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 

visualise_ICASAR_inversion(spatial_data['ifgs_dc'], S_ica, A_ica, displacement_r2['mask'], n_data = 10)
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
                   "hdbscan_param" : (35, 10),                        
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

### 相关名词或参量名或涉及的算法（不分先后）
* 名词
  * *增量变形数据*
  * *掩膜Mask*  
      掩膜是在图像或数据处理中常用的一种技术，用于指定特定区域或像素的有效性。它是一个与原始图像或数据具有相同大小的二进制图像或布尔数组，其中每个像素或元素的值表示其有效性。  
      在遥感图像处理中，掩膜通常用于指定感兴趣区域（Region of Interest, ROI），即我们只对该区域内的像素进行分析和处理，而忽略其他区域。通过将不感兴趣的区域标记为无效，可以排除这些区域对分析结果的影响，提高处理的准确性和效率。
  * *数字高程模型DEM*
    DEM是一种用于表示地球表面的高程信息的数字化模型。
  * *聚类算法HDBSCAN*
  * *流形学习算法t-SNE*
  * 
* 参量名
  * *S_ica*  
    S_ica 是 sICA 算法得到的稀疏独立成分（sparse independent components）。
  * *A_ica*  
    A_ica 是 sICA 算法得到的混合矩阵（mixing matrix）。
  * *Iq*  
    Iq 是 Q 矩阵，用于计算深度学习相关性。
  * *x_train_residual_ts*  
    x_train_residual_ts 是训练集的残差时间序列。
  * *n_clusters*  
    n_clusters 是聚类数目。
  * *S_all_info*  
    S_all_info 包含了所有独立源的信息。
  * *phUnw_mean*  
    phUnw_mean 是无包裹相位均值。
  * *sources_labels*
    sources_labels 是源标签。
* 涉及的算法

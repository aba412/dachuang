<span style="font-size: 100px;">概述</span>  

example_spatial_02的代码主要展示了如何使用 ICASAR 处理 LiCSBAS 输出的数据  
包括载入 LiCSBAS 的 .h5 文件，以及使用 ICASAR 进行独立成分分析（ICA）来提取地表变形信号。具体包括了对增量干涉图和累积干涉图进行不同的处理方式，以及展示了如何可视化拟合干涉图的结果

#%% Prep: load some LiCSBAS data 

LiCSBAS_out_folder_campi_flegrei = Path('./campi_flegrei_LiCSBAS_example_data/')                               
print(f"Opening the LiCSBAS .h5 file...", end = '')
displacement_r2, tbaseline_info, ref_xy = LiCSBAS_to_ICASAR(LiCSBAS_out_folder_campi_flegrei, figures=True)        
print(f"Done.  ")

spatial_data = {'ifgs_dc'        : displacement_r2['incremental'],
                'mask'           : displacement_r2['mask'],
                'ifg_dates_dc'   : tbaseline_info['ifg_dates'],                             
                'dem'            : displacement_r2['dem'],                                  
                'lons'           : displacement_r2['lons'],
                'lats'           : displacement_r2['lats']}

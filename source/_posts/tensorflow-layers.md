---
title: tensorflow layers
date: 2019-05-17 16:47:50
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.layers
这个模块定义在tf.contrib.layers中。主要是构建神经网络，正则化和summaries等op。它包括2个模块，2个类，以及一系列函数。

## 模块
### feature_column


### summaries
summary的工具函数定义

## 类
### class GDN
### class RevBlock

## 函数
### 常用函数
- softmax(...)
- avg_pool2d(...)
- conv2d(...)
- conv2d_transpose(...)
- convolution(...)
- convolution2d(...)
- convolution2d_transpose(...)
- dense_to_sparse(...)
- dropout(...)
- flatten(...)
- fully_connected(...)
- l1_regularizer(...)
- l2_regularizer(...)
- max_pool2d(...)
- max_pool3d(...)
- optimize_loss(...)
- xavier_initializer(...)

### 所有函数
- apply_regularization(...)
- avg_pool2d(...)
- avg_pool3d(...)
- batch_norm(...)
- bias_add(...)
- bow_encoder(...)
- bucketized_column(...)
- check_feature_columns(...)
- conv1d(...)
- conv2d(...)
- conv2d_in_plane(...)
- conv2d_transpose(...)
- conv3d(...)
- conv3d_transpose(...)
- convolution(...)
- convolution1d(...)
- convolution2d(...)
- convolution2d_in_plane(...)
- convolution2d_transpose(...)
- convolution3d(...)
- convolution3d_transpose(...)
- create_feature_spec_for_parsing(...)
- crossed_column(...)
- dense_to_sparse(...)
- dropout(...)
- embed_sequence(...)
- embedding_column(...)
- embedding_lookup_unique(...)
- flatten(...)
- fully_connected(...)
- gdn(...)
- group_norm(...)
- images_to_sequence(...)
- infer_real_valued_columns(...)
- input_from_feature_columns(...)
- instance_norm(...)
- joint_weighted_sum_from_feature_columns(...)
- l1_l2_regularizer(...)
- l1_regularizer(...)
- l2_regularizer(...)
- layer_norm(...)
- legacy_fully_connected(...)
- make_place_holder_tensors_for_base_features(...)
- max_pool2d(...)
- max_pool3d(...)
- maxout(...)
- multi_class_target(...)
- one_hot_column(...)
- one_hot_encoding(...)
- optimize_loss(...)
- parse_feature_columns_from_examples(...)
- parse_feature_columns_from_sequence_examples(...)
- real_valued_column(...)
- recompute_grad(...)
- regression_target(...)
- repeat(...)
- rev_block(...)
- safe_embedding_lookup_sparse(...)
- scattered_embedding_column(...)
- separable_conv2d(...)
- separable_convolution2d(...)
- sequence_input_from_feature_columns(...)
- sequence_to_images(...)
- shared_embedding_columns(...)
- softmax(...)
- sparse_column_with_hash_bucket(...)
- sparse_column_with_integerized_feature(...)
- sparse_column_with_keys(...)
- sparse_column_with_vocabulary_file(...)
- spatial_softmax(...)
- stack(...)
- sum_regularizer(...)
- summarize_activation(...)
- summarize_activations(...)
- summarize_collection(...)
- summarize_tensor(...)
- summarize_tensors(...)
- transform_features(...)
- unit_norm(...)
- variance_scaling_initializer(...)
- weighted_sparse_column(...)
- weighted_sum_from_feature_columns(...)
- xavier_initializer(...)
- xavier_initializer_conv2d(...)

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/contrib/layers
2.https://www.tensorflow.org/api_docs/python/tf/contrib/layers/feature_column
3.https://www.tensorflow.org/api_docs/python/tf/contrib/layers/summaries
